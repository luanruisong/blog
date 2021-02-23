---
title: "sqlbuilder（二）"
date: 2021-02-18T13:14:01+08:00
description: "Golang 基于database/sql 包封装一个自用的数据库驱动 -- 自动insert"
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
    - "insert"
---

### 序

年初七，复工第一天，带着激动的心情，打工人重返岗位

![我很好](http://blog-img.luanruisong.com/blog/img/20210218181614.png)

经过简单的忙碌，工作之余，准备小摸他一手鱼

![摸鱼](http://blog-img.luanruisong.com/blog/img/20210218181814.png)

所以，今天继续填坑

顺便上一个文章合集 [[SqlDriver]](/tags/borm/)

### 目标

- 终极目标

> 自动insert

- 过程

> 拆解一个未知的struct，实现自动化插入

### 磨刀霍霍向猪羊

工欲善其事，必先利其器。所以我们要先把需要的工具锋利起来

![磨刀](http://blog-img.luanruisong.com/blog/img/20210218193244.png)

这里我们加入了一个stringx包，用于保存我们所有的字符串操作

```go
    github.com/luanruisong/borm/stringx
```

目前用到了的函数只有一个 蛇形命名

```go
func SnakeName(name string) string {
    oldName := []rune(name)
    newName := make([]rune, 0)
    for i, v := range oldName {
        x := v
        if v < 91 {
            x += 32
            if i > 0 {
                newName = append(newName, 95)
            }
        }
        newName = append(newName, x)
    }
    return string(newName)
}

```

同时我们也自定义了一个reflectx包，用于保存我们所有的反射相关操作

```go
    github.com/luanruisong/borm/reflectx
```

下面是本次为了组织sql加入的函数

```go

//直接获取struct的type
func TypeOfStruct(i interface{}) reflect.Type {
    t := reflect.TypeOf(i)
    if t.Kind() == reflect.Ptr {
        t = t.Elem()
    }
    return t
}

//直接获取struct的value
func ValueOfStruct(i interface{}) reflect.Value {
    t := reflect.ValueOf(i)
    if t.Kind() == reflect.Ptr {
        t = t.Elem()
    }
    return t
}

//直接获取struct的name
func StructName(i interface{}) string {
    return TypeOfStruct(i).Name()
}

// 循环遍历struct的属性，使用handler处理相对应的属性
func StructRange(i interface{}, handler func(t reflect.StructField, v reflect.Value) error) error {
    v := ValueOfStruct(i)
    t := v.Type()
    for i := 0; i < t.NumField(); i++ {
        var (
            fv = v.Field(i)
            ft = t.Field(i)
        )
        if fv.CanInterface() { //过滤不可访问的属性
            if err := handler(ft, fv); err != nil {
                return err
            }
        }
    }
    return nil
}

//判断某一属性是否为空值
func IsNull(v reflect.Value) bool {
    switch v.Kind() {
    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64,
        reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64,
        reflect.Float32, reflect.Float64:
        return v.IsZero()
    case reflect.String:
        return v.String() == ""
    }
    return false
}
```

最后我们加入了一个接口，用于接收自定义表名

```go
type(
    Table interface {
        TableName() string
    }
)
```

### AutoInsert（~~杀猪~~）

刀磨完了，可以开始杀猪了！

![40米](http://blog-img.luanruisong.com/blog/img/20210218193515.png)

对于结构体field这里，我们加入一个tag叫做db（~~udb大哥再次不要打我，我起名困难~~）

然后我们在我们的sqlbuilder/insert.go中加入代码

```go

//自动插入结构体
func AutoInsert(i interface{}) (SqlBuilder, error) {
    //处理tableName 如果实现了table接口，按照接口
    var tName string
    if t, ok := i.(Table); ok {
        tName = t.TableName()
    } else {
        //未实现接口，取struct名称的蛇形
        tName = stringx.SnakeName(reflectx.StructName(i))
    }
    //如果是匿名struct，无table 返回找不到tableName
    if len(tName) == 0 {
        return nil, errors.New("can not find table name")
    }
    //初始化一个sqlBuilder
    sb := InsertInto(tName)
    //定义一个计数器
    ic := 0
    //使用reflectx的range函数，对参结构体进行遍历
    //错误可直接忽略（因为没有产生错误的地方）
    _ = reflectx.StructRange(i, func(t reflect.StructField, v reflect.Value) error {
        //获取tag，也就是自定义的column
        column := t.Tag.Get("db")
        if len(column) == 0 {
            //入无自定义column，取field名称的蛇形
            column = stringx.SnakeName(t.Name)
        }
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
    return sb, nil
}
```

老样子，写一下testcase

```go

func TestAutoInsert(t *testing.T) {
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

    autoInsert(t, s)
    autoInsert(t, s1)
    autoInsert(t, s2)
}

func autoInsert(t *testing.T, s interface{}) {
    sql, err := AutoInsert(s)
    if err != nil {
        t.Error(err.Error())
        return
    } else {
        t.Log(sql.Sql(), sql.Args())
    }
}
```

基本上测试到了 匿名结构体，不可访问属性，为空属性 等的几个维度

下面是testCase的运行结果

```go
=== RUN   TestAutoInsert
    sql_test.go:79: can not find table name
    sql_test.go:79: no column field to sql
    sql_test.go:82: insert into test_table set a = ?,bbb_bb = ?,ccc_cc = ? [asdasdasdasdas 200 3.14]
--- FAIL: TestAutoInsert (0.00s)
FAIL
```

至此，我们的autoInsert函数基本上就告一段落了

函数功能/流程如下

***结构体 -> 根据接口实现或蛇形命名构建sqlBuilder -> 获取属性 ->  根据可用属性进行insert语句构建 -> 生成sql与获取args***

一个简单却完美的功能就这样在我们的指尖实现了

![过奖](http://blog-img.luanruisong.com/blog/img/20210218192621.png)

### 后记

秉承着得过且过的核心思想，我们的注释写的比较随意，很多地方也没有写注释

![得过且过](http://blog-img.luanruisong.com/blog/img/20210218192915.png)

但是提起程序员最讨厌的四件事嘛

1. 写注释
2. 写文档
3. 别人不写注释
4. 别人不写文档

有能耐你打我啊

![打我啊](http://blog-img.luanruisong.com/blog/img/20210218193038.png)
