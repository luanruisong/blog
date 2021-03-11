---
title: "sqlbuilder（完）"
date: 2021-03-11T13:33:29+08:00
description: "Golang 基于database/sql 包封装一个自用的数据库驱动 -- sql完结篇"
categories:
    - "Development"
tags:
    - "Golang"
    - "Reflact"
    - "database"
    - "ORM"
    - "BORM"
keywords:
    - "Golang"
    - "Go"
    - "Reflact"
    - "sqlDriver"
    - "database"
    - "数据库"
    - "数据库驱动"
    - "SQL"
    - "ORM"
    - "update"
---

### 序

前几天抽空回了一趟老家，来来回回累的够呛，今天终于又回到了我热爱的工作岗位

![上班](https://blog-img.luanruisong.com/blog/img/20210311133443.jpg)

本来今天就准备开始orm的迭代器部分，但是简单的codeReview了一下，发现了两个问题

- where部分对于select语句的支持还没有写
- pk的这种方式有很大的局限性

![擦泪](https://blog-img.luanruisong.com/blog/img/20210311133830.jpg)

所以，今天准备把where的自动语句生成，重写一下，并给我们的sqlBuilder 好好的收一个尾。

![秃顶](https://blog-img.luanruisong.com/blog/img/20210311133634.jpg)

### 目标

1. 重构autoWhere函数
2. 使autoWhere支持select语句

### 准备

我们先记录一下目前的where函数

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

目前的where函数是直接解析了所有的非空字段，这样我们HalfAutoUpdate的时候，就迫使他使用了一个单独的结构体来进行where语句的生成

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

这样是非常不合理的，所以我们新加一个tag明明为『dbw』,值我们来做几个枚举值

```go
const (
    DbwTag         = "dbw"
    DbwGreaterThan = "gt"
    DbwLessThan    = "lt"
    DbwEqual       = "eq"
)
```

暂时就支持大于，小于，等于好了

![B数](https://blog-img.luanruisong.com/blog/img/20210311135011.png)

### update 开始

先加入符号转换

```go
func whereFlag(flag string) string {
    switch flag {
    case DbwEqual:
        return "="
    case DbwGreaterThan:
        return ">"
    case DbwLessThan:
        return "<"
    default:
        panic("where flag " + flag + " not support")
    }
}
```

再加入where单个sql构建的函数

```go
func whereSql(t reflect.StructField) string {
    where := t.Tag.Get(DbwTag)
    if len(where) == 0 {
        return ""
    }
    column := ColumnName(t)
    return fmt.Sprintf("%s %s ?", column, whereFlag(where))
}
```

然后再用我们的structRange 扩展成为针对于结构体的函数

```go
func AutoWhere(i interface{}) (where string, whereArgs []interface{}) {
    var (
        whereList []string
    )
    _ = reflectx.StructRange(i, func(t reflect.StructField, v reflect.Value) error {
        if sql := whereSql(t); len(sql) > 0 {
            whereList = append(whereList, sql)
            whereArgs = append(whereArgs, v.Interface())
        }
        return nil
    })
    where = strings.Join(whereList, " and ")
    return
}
```

这时候，我们舍弃掉以前的pk，AutoUpdate里面更改为，解析结构体进行where条件的构造

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
        if ws := whereSql(t); len(ws) > 0 {
            sb.Where(ws, v.Interface())
        } else {
            //获取tag，也就是自定义的column
            column := ColumnName(t)
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

重新修改一下testCase

```go
func TestAutoUpdate(t *testing.T) {
    type TestTable struct {
        AaaAa string `db:"a"`
        BbbBb int
        CccCc float64
        asda  bool
        Id    string `dbw:"gt"`
        Ts    int64  `dbw:"lt"`
    }
    s := TestTable{
        AaaAa: "asdasdasdasdas",
        BbbBb: 200,
        CccCc: 3.14,
        asda:  true,
        Id:    "10000",
        Ts:    time.Now().Unix(),
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

执行结果

```go
=== RUN   TestAutoUpdate
    sql_test.go:108: update from test_table set ccc_cc = ?,a = ?,bbb_bb = ? where ts < ? [3.14 asdasdasdasdas 200 1615442947]
--- PASS: TestAutoUpdate (0.00s)
PASS
```

可以看到，id的这个where条件被吞掉了，回去反查了一下，是因为我的update结构比较简单，再Where的时候简单的覆盖了当前的where条件导致的

```go
func (is *updateBuilder) Where(sql string, value ...interface{}) SqlBuilder {
    is.whereSql = sql
    is.whereArgs = value
    return is
}
```

![咸鱼](https://blog-img.luanruisong.com/blog/img/20210311141554.png)

那么这个时候，是时候重构一下我们的Where相关代码了（~~早就看他不顺眼了~~）

![火大](https://blog-img.luanruisong.com/blog/img/20210311141727.png)

### 重构，重构与重构

首先我们重构一下我们的接口

```go

type (
    Table interface {
        TableName() string
    }

    Sql interface {
        Sql() string
        Args() []interface{}
    }

    InsertBuilder interface {
        Sql
        From(tableName string) InsertBuilder
        Set(key string, value interface{}) InsertBuilder
    }

    DeleteBuilder interface {
        Sql
        From(tableName string) DeleteBuilder
        Where(sql string, value ...interface{}) DeleteBuilder
        And(sql string, value ...interface{}) DeleteBuilder
        Or(sql string, value ...interface{}) DeleteBuilder
    }

    UpdateBuilder interface {
        Sql
        From(tableName string) UpdateBuilder
        Set(key string, value interface{}) UpdateBuilder
        Where(sql string, value ...interface{}) UpdateBuilder
        And(sql string, value ...interface{}) UpdateBuilder
    }

    Selector interface {
        Sql
        Where(sql string, value ...interface{}) Selector
        And(sql string, value ...interface{}) Selector
        Or(sql string, value ...interface{}) Selector
        From(tableName string) Selector
        Select(...string) Selector
        OrderBy(string) Selector
        GroupBy(string) Selector
        Limit(int64) Selector
        Offset(int64) Selector
    }
)
```

重构以后，加入了where存储结构，兼容And/Or 作为where条件

```go

type (
    whereType          uint8
    singleWhereBuilder struct {
        t    whereType
        Sql  string
        Args []interface{}
    }
    whereBuilder struct {
        w []singleWhereBuilder
    }
)

const (
    whereTypeWhere = whereType(0)
    whereTypeAnd   = whereType(1)
    whereTypeOr    = whereType(2)
)

func (wb whereBuilder) Empty() bool {
    return len(wb.w) == 0
}

func (wb *whereBuilder) And(sql string, i []interface{}) {
    wb.w = append(wb.w, singleWhereBuilder{
        whereTypeAnd,
        sql,
        i,
    })
}

func (wb *whereBuilder) Or(sql string, i []interface{}) {
    wb.w = append(wb.w, singleWhereBuilder{
        whereTypeOr,
        sql,
        i,
    })
}

func (wb *whereBuilder) Where(sql string, i []interface{}) {
    wb.w = append(wb.w, singleWhereBuilder{
        whereTypeWhere,
        sql,
        i,
    })
}

func (wb whereBuilder) Sql() string {
    ret := make([]string, 0)
    for i, v := range wb.w {
        if i != 0 {
            switch v.t {
            case whereTypeAnd:
                ret = append(ret, "and")
            case whereTypeOr:
                ret = append(ret, "or")
            }
        }
        ret = append(ret, v.Sql)
    }
    return strings.Join(ret, " ")
}

func (wb whereBuilder) Args() []interface{} {
    ret := make([]interface{}, 0)
    for _, v := range wb.w {
        ret = append(ret, v.Args...)
    }
    return ret
}
```

由于接口的重构幅度比较大，接口实现就先不贴出来了,有兴趣的同学可以去github自行查找

重构以后，我们再跑下autoUpdate的testCase

```go
=== RUN   TestAutoUpdate
    sql_test.go:108: update from test_table set ccc_cc = ?,a = ?,bbb_bb = ? where id > ? and ts < ? [3.14 asdasdasdasdas 200 10000 1615446491]
--- PASS: TestAutoUpdate (0.00s)
PASS
```

完全ok，可以看到，这里的条件已经完美的兼容，并且是可以支持 大于，小于，等于三种条件

![吊](https://blog-img.luanruisong.com/blog/img/20210311151015.jpg)

### 拖了许久的select

既然自动where已经写完，基本上自动select的代码已经呼之欲出了

```go

func AutoSelect(i interface{}) (Selector, error) {
    tName := TableName(i)
    if len(tName) == 0 {
        return nil, errors.New("can not find table name")
    }
    sb := SelectFrom(tName)

    ic := 0 //有效列计数
    _ = reflectx.StructRange(i, func(t reflect.StructField, v reflect.Value) error {
        if ws := whereSql(t); len(ws) > 0 {
            sb.And(ws, v.Interface())
        } else {
            //获取tag，也就是自定义的column
            column := ColumnName(t)
            // "-" 表示忽略，空数据 也直接跳过
            if column == "-" || reflectx.IsNull(v) {
                return nil
            }
            // 书接上回，进行sql构建（单一属性）
            sb.Select(column)
            ic++
        }
        return nil
    })
    //计数器为0表示无可用列构建，处理 *查询
    if ic == 0 {
        sb.Select("*")
    }
    return sb, nil
}
```

老样子，testCase

```go

type TestSelectTable struct {
    AaaAa string `db:"a"`
    BbbBb int
    CccCc float64
    asda  bool
    Id    string `dbw:"gt"`
    Ts    int64  `dbw:"lt"`
}

func (TestSelectTable) TableName() string {
    return "asd"
}
func TestAutoSelect(t *testing.T) {
    s := TestSelectTable{
        AaaAa: "asdasdasdasdas",
        BbbBb: 200,
        CccCc: 3.14,
        asda:  true,
        Id:    "10000",
        Ts:    time.Now().Unix(),
    }
    sql, err := AutoSelect(s)
    if err != nil {
        t.Error(err.Error())
        return
    } else {
        t.Log(sql.Sql(), sql.Args())
    }
}
```

接着，上运行结果

```go
=== RUN   TestAutoSelect
    sql_test.go:139: select a,bbb_bb,ccc_cc from asd where id > ? and ts < ?     [10000 1615447313]
--- PASS: TestAutoSelect (0.00s)
PASS
```

至此，有关sql构建的代码，全部完结，下一期就开始迭代器相关的部分，也是主要针对 select的处理

第一部分已经完结，想夸我的小伙伴，可以不用憋着了

![优秀](https://blog-img.luanruisong.com/blog/img/20210311152358.jpg)
