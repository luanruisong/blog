---
title: "sqlbuilder（一）"
date: 2021-02-12T13:14:01+08:00
description: "Golang 基于database/sql 包封装一个自用的数据库驱动 -- SQL 生成器"
categories:
    - "Development"
tags:
    - "Golang"
    - "Reflact"
    - "database"
    - "ORM"
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
---

### 序

大年初一，在这给各位拜年啦！

![拜年](http://blog-img.luanruisong.com/blog/img/20210212181901.png)

由于就地过年实在无聊，所以大年初一准备先小填一下自己挖的坑。

今天咱们就先来聊聊我们的sql生成器，我准备叫他sqlBuilder（~~起名太难了UDB大哥别打我~~）

![挨打](http://blog-img.luanruisong.com/blog/img/20210212182720.png)

文章代码都会收录再 [gitbug](https://github.com/luanruisong/borm)当中，想看看的同学请自取

另外，以减少麻烦为第一原则，我们本着mysql是第一生产力（~~就是懒得处理方言~~）的原则，本文所有示例均以mysql为准。

复杂sql也不在我们本次讨论范围之内，我们的目标是借由 orm的方式探究更高级更优雅的golang编码，而不需要拘泥于完整的orm框架实现

### 目标

首先，给我们的sqlbuilder定下几个基础目标，具体目标如下

- 构建一个sqlBuild结构体用于生成最终的sql与args
  
- 完成标准sql的生成，包含语句与参数

- 完成公用struct的语句生成，包含(insert/update/delete)

### sqlBuilder

首先，我们来给我们的sqlbuild定义几个接口，用于对外的使用

```go

type (

    Sql interface {
        Sql() string
        Args() []interface{}
    }

    SqlBuilder interface {
        Sql
        From(tableName string) SqlBuilder
        Set(key string, value interface{}) SqlBuilder
        Where(sql string, value ...interface{}) SqlBuilder
    }

    Selector interface {
        Sql
        From(tableName string) Selector
        Where(sql string, value ...interface{}) Selector
        Select(...string) Selector
        OrderBy(string) Selector
        GroupBy(string) Selector
        Limit(int64) Selector
        Offset(int64) Selector
    }
)

```

1. Sql 不用多说，用于最终的sql和参数的获取

2. SqlBuilder 用于基础insert/update/delete 语句的组装

3. Selector 这个就比较麻烦了，由于查询语句比较复杂，这里定义了基本的查询结构，Selector 也是用于查询语句的组装

实现的代码可以自行再github项目中查看

接下来我们写一下单元测试，源码在这 

```go

import (
    "testing"
    "time"
)

func TestInsert(t *testing.T) {
    sql := InsertInto("table")
    sql.Set("a", 1).
        Set("b", true).
        Set("c", "true")
    t.Log(sql.Sql())
    t.Log(sql.Args())
}

func TestUpdate(t *testing.T) {
    sql := UpdateFrom("table")
    sql.Set("a", 1).
        Set("b", true).
        Set("c", "true")

    sql.Where("id=?", 1)
    t.Log(sql.Sql())
    t.Log(sql.Args())
}

func TestDelete(t *testing.T) {
    sql := DeleteFrom("table")

    sql.Where("id=?", 1)
    t.Log(sql.Sql())
    t.Log(sql.Args())
}

func TestSelect(t *testing.T) {
    sql := SelectFrom("table")
    sql.Select("a", "b", "c").
        Where("id=? and time = ?", 1, time.Now()).
        GroupBy("g1").
        OrderBy("o1").
        Limit(10).
        Offset(200)
    t.Log(sql.Sql())
    t.Log(sql.Args())
}
```

单元测试的运行结果

```shell

=== RUN   TestInsert
    sql_test.go:13: insert into table set a = ?,b = ?,c = ?
    sql_test.go:14: [1 true true]
--- PASS: TestInsert (0.00s)

=== RUN   TestUpdate
    sql_test.go:24: update from table set a = ?,b = ?,c = ? where id=?
    sql_test.go:25: [1 true true 1]
--- PASS: TestUpdate (0.00s)

=== RUN   TestDelete
    sql_test.go:32: delete from table where id=?
    sql_test.go:33: [1]
--- PASS: TestDelete (0.00s)

=== RUN   TestSelect
    sql_test.go:44: select a,b,c from table where id=? and time = ? group by g1 order by o1 limit 10 offset 200
    sql_test.go:45: [1 2021-02-12 21:27:14.328956 +0800 CST m=+0.000700157]
--- PASS: TestSelect (0.00s)
PASS
Process finished with exit code 0
```

这样，基本的sql生成器就完成了，虽然还有很多不足之处，但是基础的CRUD操作已经满足了。

至此，我们第一篇关于sqlBuilder的文章到此就结束了呢。

![敬礼](http://blog-img.luanruisong.com/blog/img/20210212213305.png)
