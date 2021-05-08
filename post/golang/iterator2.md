---
title: "iterator (补充)"
date: 2021-05-08T16:36:56+08:00
description: "Golang 基于database/sql 包封装一个自用的数据库驱动 -- 数据迭代器"
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

之前再写完迭代器代码之后，有一个部分让我辗转反侧，夜不能寐

![bb](https://blog-img.luanruisong.com/blog/img/20210508155633.png)

最终下定决心，再最后整合之前，还是把这个问题先处理掉，顺便水一篇blog

### 案发现场

关于单次迭代里，处理结构体的时候，对于time这个类型不友好，我自己也是知道的

我们再来看下单次迭代sql.Rows的代码

```go
//fetchResult 通过列名抓取响应属性生成一个类型指针
func fetchResult(rows *sql.Rows, itemT reflect.Type, columns []string) (reflect.Value, error) {
    var err error
    objT := reflect.New(itemT) //创建一个薪的指针
    values := make([]interface{}, len(columns)) //根据查询列明构建需要获取的field指针切片
    fieldMap, _ := reflectx.StructMap(objT.Interface())//根据反射获取列名与具体field的映射
    tmpMap := make(map[string]reflect.Value)//创建一个临时map 用于处理time的赋值
    //根据查询列明循环
    for i, k := range columns {
        f, ok := fieldMap[k]//查找当前类型的field映射
        if !ok {//不存在，加入占位interface
            values[i] = new(interface{})
            continue
        }
        values[i] = f.Addr().Interface()
        if ok {
            //识别如果是time类型特殊处理，并使用映射map来保存具体值
            switch values[i].(type) {
            case time.Time, *time.Time:
                tmpValue := reflect.New(reflect.TypeOf(""))
                values[i] = tmpValue.Interface()
                tmpMap[k] = tmpValue
            }
        }
    }
    //调用scan函数
    err = rows.Scan(values...)
    if err == nil {
        //遍历我们的time映射，赋值回具体的field
        for k, v := range tmpMap {
            curr := v.Elem().String()
            t, _ := time.Parse("2006-01-02 15:04:05", curr)
            currV := reflect.ValueOf(t)
            fieldMap[k].Set(currV)
        }
    }
    //返回当前obj
    return objT, err
}
```

### 杀人冻鸡

先来说一下，All和One 函数迭代的整个流程

![iter](https://blog-img.luanruisong.com/blog/img/20210508161345.png)

但是再这个流程的column对应filed切片的组装上，我们目前是用的是这个方式

![jb](https://blog-img.luanruisong.com/blog/img/20210508162904.png)

这明显是一个比较蠢的实现方式

![chun](https://blog-img.luanruisong.com/blog/img/20210508163009.png)

翻了翻源码，发现sql包提供了一个Scanner接口，专门用于这种特殊类型的情况，所以我们的实现流程修改成了以下形式

![bj2](https://blog-img.luanruisong.com/blog/img/20210508163116.png)

这么一看，最起码流程上，优雅的多了嘛

### 结案陈词

先上我们的TimeScanner代码

```go
type (
    TimeScanner struct {
        v      reflect.Value
        format string
    }
)

const defTimeFormat = "2006-01-02 15:04:05"

func (t TimeScanner) Scan(src interface{}) (err error) {
    var (
        curr time.Time
    )
    switch src.(type) {
    case []byte:
        curr, err = time.Parse(t.format, string(src.([]byte)))
        if err != nil {
            return
        }
    case string:
        curr, err = time.Parse(t.format, src.(string))
        if err != nil {
            return
        }
    case int64:
        curr = time.Unix(src.(int64), 0)
    case nil:
        return nil
    default:
        return ErrTimeScan
    }
    currV := reflect.ValueOf(curr)
    t.v.Set(currV)
    return nil
}

func NewTimeScanner(v reflect.Value, format string) sql.Scanner {
    return &TimeScanner{
        v:      v,
        format: format,
    }
}
```

这里使用自定义结构体，实现sql/Scanner 的接口

接下来是调用部分的函数

```go
//fetchResult 通过列名抓取响应属性生成一个类型指针
func fetchResult(rows *sql.Rows, itemT reflect.Type, columns []string) (reflect.Value, error) {
    objT := reflect.New(itemT)
    values := make([]interface{}, len(columns))
    fieldMap, _ := reflectx.StructMap(objT.Interface())
    for i, k := range columns {
        f, ok := fieldMap[k]
        if !ok {
            values[i] = new(interface{})
            continue
        }
        curr := f.Addr().Interface()
        switch curr.(type) {
        case time.Time, *time.Time:
            format := defTimeFormat
            if dbFmt := f.Tag.Get("fmt"); len(dbFmt) > 0 {
                format = dbFmt
            }
            values[i] = NewTimeScanner(f.Value, format)
        default:
            values[i] = curr
        }
    }
    return objT, rows.Scan(values...)
}
```

可以看到，这里的逻辑明显清晰了很多

- 去掉了让人头疼的映射关系
- 不需要再scan之后还要反查time属性进行赋值
- 还顺便加入了根据fmt这个tag 可以自己定义time结构

### 完结撒花

至此，本次由于当时脑瘫引起的代码优化就圆满结束了呢

![sj](https://blog-img.luanruisong.com/blog/img/20210508163835.png)
