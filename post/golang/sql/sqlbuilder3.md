---
title: "sqlbuilder（三）"
date: 2021-02-23T13:14:01+08:00
description: "Golang 基于database/sql 包封装一个自用的数据库驱动 -- 自动update"
categories:
    - "Development"
tags:
    - "Golang"
    - "Reflect"
    - "database"
    - "ORM"
    - "BORM"
keywords:
    - "Golang"
    - "Go"
    - "Reflect"
    - "sqlDriver"
    - "database"
    - "数据库"
    - "数据库驱动"
    - "SQL"
    - "ORM"
    - "update"
---

### 序

复工回来，除了第一天比较清闲之外，后续的工作稍显忙碌

![忙](https://blog-img.luanruisong.com/blog/img/20210223164942.gif)

忙碌的需求告一段落，今天继续来整个活

![整活](https://blog-img.luanruisong.com/blog/img/20210223165127.png)

老样子，先上个文章合集 [[SqlDriver]](/tags/borm/)

今天呢，准备把sql生成这里收收尾，处理一下自动的update和select 参数生成

### 目标

- 终极目标

> 1. 自动 update struct 填入 set 部分
>
> 2. 自动拆解一个 struct 转换成为where条件
>
> 3. 自动 一个带有pk的结构体
>

- 过程

> - 拆解struct 实现update语句转化
> - 拆解struct 实现where条件转化
> - 定义新tag，识别出主键，进行完全自动的update
>

### 磨刀

老样子，先磨刀
![磨刀](https://blog-img.luanruisong.com/blog/img/20210223170301.png)

先把根据结构体获取表名的函数提取出来，这里面用到的地方会比较多

```go
func TableName(i interface{}) string {
    var tName string
    if t, ok := i.(Table); ok {
        tName = t.TableName()
    } else {
        //未实现接口，取struct名称的蛇形
        tName = stringx.SnakeName(reflectx.StructName(i))
    }
    return tName
}

```

接着提取一下获取列名的函数

```go
func ColumnName(t reflect.StructField) string {
    column := t.Tag.Get("db")
    if len(column) == 0 {
        //如无自定义column，取field名称的蛇形
        column = stringx.SnakeName(t.Name)
    }
    return column
}
```

接下来是testCase

```go

func TestTableName(t *testing.T) {
    s := struct {
        AaaAa string `db:"a"`
        BbbBb int
        CccCc float64
    }{}

    type TestTable struct {
        AaaAa string `db:"a"`
        BbbBb int
        CccCc float64
        asda  bool
    }

    s1 := TestTable{}
    s2 := TestTable{
        AaaAa: "asdasdasdasdas",
        BbbBb: 200,
        CccCc: 3.14,
        asda:  true,
    }

    t.Log(TableName(s))
    t.Log(TableName(s1))
    t.Log(TableName(s2))
}

func TestColumnName(t *testing.T) {
    s := struct {
        AaaAa string `db:"a"`
        BbbBb int
        CccCc float64
    }{}

    reflectx.StructRange(s, func(f reflect.StructField, v reflect.Value) error {
        t.Log(ColumnName(f))
        return nil
    })
}

```

以及，testCase的运行结果

```go
=== RUN   TestTableName
    utils_test.go:32: 
    utils_test.go:33: test_table
    utils_test.go:34: test_table
--- PASS: TestTableName (0.00s)
PASS
=== RUN   TestColumnName
    utils_test.go:45: a
    utils_test.go:45: bbb_bb
    utils_test.go:45: ccc_cc
--- PASS: TestColumnName (0.00s)
PASS
```

自动拆解struct的工具我们基本上已经写好了，但是从struct到Where的转换，目前还是没有

并且由于 struct到where条件的转换，在本次的三个目标都比较重要，所以放在磨刀环节

上代码

```go

func StructToWhere(i interface{}) (where string, whereArgs []interface{}) {
    var (
        whereList []string
    )
    _ = reflectx.StructRange(i, func(t reflect.StructField, v reflect.Value) error {
        //获取tag，也就是自定义的column
        column := ColumnName(t)
        // "-" 表示忽略，空数据 也直接跳过
        if column == "-" || reflectx.IsNull(v) {
            return nil
        }
        whereList = append(whereList, fmt.Sprintf("%s = ?", column))
        whereArgs = append(whereArgs, v.Interface())
        return nil
    })
    where = strings.Join(whereList, " and ")
    return
}

```

继续testCase

```go
func TestStructToWhere(t *testing.T) {
    s := struct {
        AaaAa string `db:"a"`
        BbbBb int
        CccCc float64
    }{}

    type TestTable struct {
        AaaAa string `db:"a"`
        BbbBb int
        CccCc float64
        asda  bool
    }

    s1 := TestTable{}
    s2 := TestTable{
        AaaAa: "asdasdasdasdas",
        BbbBb: 200,
        CccCc: 3.14,
        asda:  true,
    }

    t.Log(StructToWhere(s))
    t.Log(StructToWhere(s1))
    t.Log(StructToWhere(s2))
}
```

以及，运行结果

```go
=== RUN   TestStructToWhere
    utils_test.go:73:  []
    utils_test.go:74:  []
    utils_test.go:75: a = ? and bbb_bb = ? and ccc_cc = ? [asdasdasdasdas 200 3.14]
--- PASS: TestStructToWhere (0.00s)
PASS
```

### 杀猪

首先呢，我们把半自动的update先安排上

```go
func HalfAutoUpdate(set interface{}, where interface{}) (SqlBuilder, error) {
    //老样子 先拿表名来生成一个sqlbuilder
    tName := TableName(set)
    if len(tName) == 0 {
        return nil, errors.New("can not find table name")
    }
    sb := UpdateFrom(tName)

    ic := 0 //有效列计数
    _ = reflectx.StructRange(set, func(t reflect.StructField, v reflect.Value) error {
        //获取tag，也就是自定义的column
        column := ColumnName(t)
        // "-" 表示忽略，空数据 也直接跳过
        if column == "-" || reflectx.IsNull(v) {
            return nil
        }
        // 书接上回，进行sql构建（单一属性）
        sb.Set(column, v.Interface())
        ic++
        return nil
    })
    //计数器为0表示无可用列构建，返回错误
    if ic == 0 {
        return nil, errors.New("no column field to sql")
    }
    whereSql, whereArgs := StructToWhere(where)
    if len(whereSql) > 0 {
       sb.Where(whereSql, whereArgs...)
    }
    return sb, nil
}
```

这样的半自动基本上已经ok，虽然半自动也挺好，但是我还是希望进行全自动的处理,尤其是进行了两次的结构体遍历，让人十分不爽

![不爽](https://blog-img.luanruisong.com/blog/img/20210223180750.png)

所以我准备加入一个tag 叫做pk，当我们识别到pk==1的时候 主动把他作为主键update的条件

所以在structRange的时候，获取完StructField，就开始进行分道扬镳了

![骨灰](https://blog-img.luanruisong.com/blog/img/20210223181108.png)

先加入识别pk 的函数

```go
func IsPk(t reflect.StructField) bool {
    tag := t.Tag.Get("pk")
    if len(tag) == 0 {
        //如无自定义column，取field名称的蛇形
        return false
    }
    return tag == "1"
}
```

以及这个函数的testCase

```go
func TestIsPk(t *testing.T) {
    s := struct {
        AaaAa string `db:"a"`
        BbbBb int
        CccCc float64 `pk:"1"`
    }{}

    reflectx.StructRange(s, func(f reflect.StructField, v reflect.Value) error {
        t.Log(f.Name, IsPk(f))
        return nil
    })
}
```

以及testCase的执行结果

```go
=== RUN   TestIsPk
    utils_test.go:86: AaaAa false
    utils_test.go:86: BbbBb false
    utils_test.go:86: CccCc true
--- PASS: TestIsPk (0.00s)
PASS
```

由此，我们的全自动 update 就诞生了

```go

func AutoUpdate(i interface{}) (SqlBuilder, error) {
    //老样子 先拿表名来生成一个sqlbuilder
    tName := TableName(i)
    if len(tName) == 0 {
        return nil, errors.New("can not find table name")
    }
    sb := UpdateFrom(tName)

    ic := 0 //有效列计数
    _ = reflectx.StructRange(i, func(t reflect.StructField, v reflect.Value) error {
        //获取tag，也就是自定义的column
        column := ColumnName(t)
        if IsPk(t) {
            sb.Where(fmt.Sprintf("%s = ?", column), v.Interface())
        } else {
            // "-" 表示忽略，空数据 也直接跳过
            if column == "-" || reflectx.IsNull(v) {
                return nil
            }
            // 书接上回，进行sql构建（单一属性）
            sb.Set(column, v.Interface())
            ic++
        }
        return nil
    })
    //计数器为0表示无可用列构建，返回错误
    if ic == 0 {
        return nil, errors.New("no column field to sql")
    }
    return sb, nil
}
```

然后搞一下它的testCase

```go
func TestAutoUpdate(t *testing.T) {
    type TestTable struct {
        AaaAa string `db:"a"`
        BbbBb int
        CccCc float64
        asda  bool
        Id    string `pk:"1"`
    }
    s := TestTable{
        AaaAa: "asdasdasdasdas",
        BbbBb: 200,
        CccCc: 3.14,
        asda:  true,
    }
    sql, err := AutoUpdate(s)
    if err != nil {
        t.Error(err.Error())
        return
    } else {
        t.Log(sql.Sql(), sql.Args())
    }
}
```

运行结果

```go
=== RUN   TestAutoUpdate
    sql_test.go:105: update from test_table set a = ?,bbb_bb = ?,ccc_cc = ? where id = ? [asdasdasdasdas 200 3.14 ]
--- PASS: TestAutoUpdate (0.00s)
PASS
```

### 后记

至此，今天的目标已经完成的差不多了，难得贴了这么多代码，已经正经的不像是我了

![正经](https://blog-img.luanruisong.com/blog/img/20210223182121.png)

本期应该可以说是自打使用Hugo建立bolg以来，代码贴的最多的一期了，所以今天这个活，是不是。。

![整挺好](https://blog-img.luanruisong.com/blog/img/20210223165228.png)
