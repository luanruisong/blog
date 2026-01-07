---
title: "borm  final"
date: 2021-05-28T14:51:25+08:00
description: "Golang 基于database/sql 包封装一个自用的数据库驱动 -- 完结篇"
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

风和日丽，这次也有日子没写博客了。

主要原因吧，就是因为  ↓↓↓

![CEzMxp](https://blog-img.luanruisong.com/blog/img/2021/05/28/CEzMxp.jpg)

上次整个搞定数据迭代之后，我就知道，是时候整合一下所有我们写好的功能了。

毕竟写了这么多，咋样也得出个1.0可用版不是？

### 接口封装

这部分，我们除了基础的sql操作之外，还要整合进去我们sqlbuilder的功能

所以定义了如下接口

```go
    BormExecutor interface {
        Exec() (sql.Result, error)
    }
    Updater interface {
        BormExecutor
        Set(key string, value interface{}) Updater
        Where(sql string, value ...interface{}) Updater
        And(sql string, value ...interface{}) Updater
        Or(sql string, value ...interface{}) Updater
    }
    Deleter interface {
        BormExecutor
        Where(sql string, value ...interface{}) Deleter
        And(sql string, value ...interface{}) Deleter
        Or(sql string, value ...interface{}) Deleter
    }

    Inserter interface {
        BormExecutor
        Values(interface{}) Inserter
    }
    Selector interface {
        iterator.Iterator
        Where(sql string, value ...interface{}) Selector
        AutoWhere(i interface{}) Selector
        And(sql string, value ...interface{}) Selector
        Or(sql string, value ...interface{}) Selector
        From(tableName string) Selector
        Select(...string) Selector
        OrderBy(string) Selector
        GroupBy(string) Selector
        Limit(int64) Selector
        Offset(int64) Selector
    }
```

然后抽象一下数据库相关的操作，开放出来对外的基础接口

```go
    SqlExecutor interface {
        Exec(sql string, args ...interface{}) (sql.Result, error)
        Query(sql string, args ...interface{}) (*sql.Rows, error)
        QueryRow(sql string, args ...interface{}) *sql.Row
    }
    Executor interface {
        SqlExecutor
        InsertInto(t string) Inserter
        AutoInsert(t interface{}) (sql.Result, error)
        UpdateFrom(tableName string) Updater
        AutoUpdate(i interface{}) (sql.Result, error)
        DeleteFrom(tableName string) Deleter
        AutoDelete(i interface{}) (sql.Result, error)

        Select(...string) Selector
        SelectFrom(string) Selector

        Count(i interface{}) (int64, error)
    }
    DataBase interface {
        Executor
        SetMaxOpenConns(n int)
        Close() error

        Begin() (Tx, error)
    }
    Tx interface {
        Executor
        Commit() error
        RollBack() error
    }
```

最终，我们再加入一个 sql驱动接口

```go
    Connector interface {
        DriverName() string
        ConnStr() string
        GetPoolSize() int
    }
```

这里我们的基础接口定义基本上就完成了

### 接口实现

从简到难，我们先挑代码少的来说

#### inserter

inserter部分是最简单的，基本上就是引用的sqlbuilder的insert生成sql

```go
type (
    inserter struct {
        sb   sqlbuilder.InsertBuilder
        exec SqlExecutor
    }
)

func (i *inserter) Exec() (sql.Result, error) {
    sb := i.sb
    return i.exec.Exec(sb.Sql(), sb.Args()...)
}

func (i *inserter) Values(i2 interface{}) Inserter {
    i.sb.Values(i2)
    return i
}

func NewInserter(exec SqlExecutor, sb sqlbuilder.InsertBuilder) Inserter {
    return &inserter{
        sb:   sb,
        exec: exec,
    }
}

```

#### deleter

自动删除这部分，基本是对于自动化where的一个处理，但逻辑部分也比较简单，就不多赘述了。

```go
type (
    deleter struct {
        sb   sqlbuilder.DeleteBuilder
        exec SqlExecutor
    }
)

func (i *deleter) Or(sql string, value ...interface{}) Deleter {
    i.sb.Or(sql, value...)
return i
}

func (i *deleter) Where(sql string, value ...interface{}) Deleter {
    i.sb.Where(sql, value...)
    return i
}

func (i *deleter) And(sql string, value ...interface{}) Deleter {
    i.sb.And(sql, value...)
    return i
}

func (i *deleter) Exec() (sql.Result, error) {
    sb := i.sb
    return i.exec.Exec(sb.Sql(), sb.Args()...)
}

func NewDeleter(exec SqlExecutor, sb sqlbuilder.DeleteBuilder) Deleter {
    return &deleter{
        sb:   sb,
        exec: exec,
    }
}
```

#### updater

自动修改这里，除了基础的where条件构造，还多了一个修改条件的添加。

代码如下：

```go
type (
    updater struct {
        sb   sqlbuilder.UpdateBuilder
        exec SqlExecutor
    }
)

func (i *updater) Set(key string, value interface{}) Updater {
    i.sb.Set(key, value)
    return i
}

func (i *updater) Or(sql string, value ...interface{}) Updater {
    i.sb.Or(sql, value...)
    return i
}

func (i *updater) Where(sql string, value ...interface{}) Updater {
    i.sb.Where(sql, value...)
    return i
}

func (i *updater) And(sql string, value ...interface{}) Updater {
    i.sb.And(sql, value...)
    return i
}

func (i *updater) Exec() (sql.Result, error) {
    sb := i.sb
    return i.exec.Exec(sb.Sql(), sb.Args()...)
}

func NewUpdate(exec SqlExecutor, sb sqlbuilder.UpdateBuilder) Updater {
    return &updater{
        sb:   sb,
        exec: exec,
    }
}

```

#### selector

selector这里，分位了两个部分，一个部分是对于sql的构造，另一部分是对于执行好了的结果进行迭代。

基础结构定义

```go
type (
    selector struct {
        sb   sqlbuilder.Selector
        exec SqlExecutor
    }
)


func NewSelector(exec SqlExecutor, sb sqlbuilder.Selector) Selector {
    return &selector{
        sb:   sb,
        exec: exec,
    }
}
```

##### sql构造

```go
func (s *selector) AutoWhere(i interface{}) Selector {
    if !reflectx.IsNull(i) {
        if where := sqlbuilder.AutoWhere(i); where != nil {
            s.sb.Where(where.Sql(), where.Args()...)
        }
    }
    return s
}
func (s *selector) Where(sql string, value ...interface{}) Selector {
    s.sb.Where(sql, value...)
    return s
}

func (s *selector) And(sql string, value ...interface{}) Selector {
    s.sb.And(sql, value...)
    return s
}

func (s *selector) Or(sql string, value ...interface{}) Selector {
    s.sb.Or(sql, value...)
    return s
}

func (s *selector) From(tableName string) Selector {
    s.sb.From(tableName)
    return s
}

func (s *selector) Select(s2 ...string) Selector {
    s.sb.Select(s2...)
    return s
}

func (s *selector) OrderBy(s2 string) Selector {
    s.sb.OrderBy(s2)
    return s
}

func (s *selector) GroupBy(s2 string) Selector {
    s.sb.GroupBy(s2)
    return s
}

func (s *selector) Limit(i int64) Selector {
    s.sb.Limit(i)
    return s
}

func (s *selector) Offset(i int64) Selector {
    s.sb.Offset(i)
    return s
}
```

以上也可以基本的认为是对于sqlbuilder包下的查询语句生成器的封装

接下来是第二部分

##### 迭代

```go
func (s *selector) All(i interface{}) error {
    rows, err := s.exec.Query(s.sb.Sql(), s.sb.Args()...)
    if err != nil {
        return err
    }
    iter := iterator.New(rows)
    return iter.All(i)
}

func (s *selector) One(i interface{}) error {
    sb := s.sb.Limit(1)
    rows, err := s.exec.Query(sb.Sql(), sb.Args()...)
    if err != nil {
        return err
    }
    iter := iterator.New(rows)
    return iter.One(i)
}
```

### 接口整合

光实现基础接口还是不够，还要整合到一起方便使用。

看到上面实现代码，不难发现，我们这里基于了一个接口叫SqlExecutor，他的基本职责就是相当于处理sql的最终运行。

我们这里涉及到到的SqlExecutor 只有两个

- 原生sql.DB
- 原生sql.Tx

然后基于SqlExecutor，我们实现了我们自己的 executor

#### executor

executor的职责很简单

1. 就是根据当前的SqlExecutor来执行语句
2. 根据当前的SqlExecutor创建我们之间定义的几个sql处理器
3. 临时补上的一个比较常用的Count函数

```go

type (
    executor struct {
        exec SqlExecutor
    }
)

func (d *executor) Exec(sql string, args ...interface{}) (sql.Result, error) {
    return d.exec.Exec(sql, args...)
}

func (d *executor) Query(sql string, args ...interface{}) (*sql.Rows, error) {
    return d.exec.Query(sql, args...)
}

func (d *executor) QueryRow(sql string, args ...interface{}) *sql.Row {
    return d.exec.QueryRow(sql, args...)
}

func (d *executor) Select(s ...string) Selector {
    return NewSelector(d.exec, sqlbuilder.Select(s...))
}

func (d *executor) SelectFrom(s string) Selector {
    return NewSelector(d.exec, sqlbuilder.SelectFrom(s))
}

func (d *executor) DeleteFrom(tableName string) Deleter {
    return NewDeleter(d.exec, sqlbuilder.DeleteFrom(tableName))
}

func (d *executor) AutoDelete(i interface{}) (sql.Result, error) {
    sb, err := sqlbuilder.AutoDelete(i)
    if err != nil {
        return nil, err
    }
    return NewDeleter(d.exec, sb).Exec()
}

func (d *executor) UpdateFrom(tableName string) Updater {
    return NewUpdate(d.exec, sqlbuilder.UpdateFrom(tableName))
}

func (d *executor) AutoUpdate(i interface{}) (sql.Result, error) {
    sb, err := sqlbuilder.AutoUpdate(i)
    if err != nil {
        return nil, err
    }
    return NewUpdate(d.exec, sb).Exec()
}

func (d *executor) InsertInto(t string) Inserter {
    return NewInserter(d.exec, sqlbuilder.InsertInto(t))
}

func (d *executor) AutoInsert(t interface{}) (sql.Result, error) {
    return NewInserter(d.exec, sqlbuilder.AutoInsert(t)).Exec()
}

func (d *executor) Count(i interface{}) (int64, error) {
    var (
        tb  = sqlbuilder.TableName(i)
        sb  = sqlbuilder.Select("count(*) as c").From(tb)
        tmp struct {
            C int64
        }
        err error
    )
    err = NewSelector(d.exec, sb).AutoWhere(i).One(&tmp)
    return tmp.C, err
}

func newExec(db SqlExecutor) *executor {
    return &executor{exec: db}
}

```

#### dataBase

搞定executor之后，至于dataBase的封装，就更加的简单了。

他只的职责只有3个

1. 关闭链接
2. 打开一个Tx
3. 设置当前链接的连接池数量

```go
type (
    dataBase struct {
        *executor
        db *sql.DB
    }
)

func (d *dataBase) Close() error {
    return d.db.Close()
}

func (d *dataBase) SetMaxOpenConns(n int) {
    d.db.SetMaxOpenConns(n)
}

func (d *dataBase) Begin() (Tx, error) {
    sqlTx, err := d.db.Begin()
    if err != nil {
        return nil, err
    }
    return newTx(sqlTx), nil
}
```

当然这里没提到dataBase是怎么初始化的，这个稍后再驱动部分会补上。

#### Tx

dataBase有个开启Tx的功能来实现事物，但是Tx的实现基础上跟dataBase差不多，只是扩展的函数只包含提交/回滚。

```go
type (
    tx struct {
        *executor
        db *sql.Tx
    }
)

func (d *tx) Commit() error {
    return d.db.Commit()
}

func (d *tx) RollBack() error {
    return d.db.Rollback()
}

func newTx(db *sql.Tx) Tx {
    return &tx{
        executor: newExec(db),
        db:       db,
    }
}
```

#### driver

最后呢，我们还有个根据driver来创建dataBase的函数。

整合以后的全局入口，就在这里了

```go
func NewDB(driver, connStr string) (DataBase, error) {
    db, err := sql.Open(driver, connStr)
    if err != nil {
        return nil, err
    }
    if err := db.Ping(); err != nil {
        return nil, err
    }
    return &dataBase{
        executor: newExec(db),
        db:       db,
    }, nil
}
```

### 杂项

db相关的代码已经写完了，眼睛比较贼的小伙伴应该发现，我们还有一个接口没有用到。

就是 **Connector**

这个接口是用在我们db统一初始化的地方，db包内部无需关心这些参数的问题，只需要基于最基本的sql包进行操作即可

我们在最外层的入口定义了一个borm.New函数

```go
func New(conn db.Connector) (db.DataBase, error) {
    if reflectx.IsNull(conn) {
        return nil, errors.New("nil connector")
    }
    db, err := db.NewDB(conn.DriverName(), conn.ConnStr())
    if err == nil {
        db.SetMaxOpenConns(conn.GetPoolSize())
    }
    return db, err
}
```

这个函数搞定以后，下一步我提供了一个原生的mysql接口实现，方便大家直接设置

```go
type MySQLConfig struct {
    User     string `yaml:"user" json:"user" xml:"user"`
    Pwd      string `yaml:"pwd" json:"pwd" xml:"pwd"`
    Db       string `yaml:"db" json:"db" xml:"db"`
    Host     string `yaml:"host" json:"host" xml:"host"`
    Port     int    `yaml:"port" json:"port" xml:"port"`
    PoolSize int    `yaml:"pool_size" json:"pool_size" xml:"pool_size"`
    Charset  string `yaml:"charset" json:"charset" xml:"charset"`
    Loc      string `yaml:"loc" json:"loc" xml:"loc"`
}

func (mc *MySQLConfig) DriverName() string {
    return "mysql"
}

func (mc *MySQLConfig) ConnStr() string {
    return fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=%s&parseTime=true&loc=%s", mc.User, mc.Pwd, mc.Host, mc.defPort(), mc.Db, mc.defCharset(), mc.defLoc())
}

func (mc *MySQLConfig) defPort() int {
    if mc.Port == 0 {
        mc.Port = 3306
    }
    return mc.Port
}

func (mc *MySQLConfig) GetPoolSize() int {
    if mc.PoolSize == 0 {
        mc.PoolSize = 10
    }
    return mc.PoolSize
}

func (mc *MySQLConfig) defCharset() string {
    if len(mc.Charset) == 0 {
        mc.Charset = "utf8"
    }
    return mc.Charset
}

func (mc *MySQLConfig) defLoc() string {
    if len(mc.Loc) == 0 {
        mc.Loc = "Asia%2FShanghai"
    }
    return mc.Loc
}
```

基础设置与默认值也都在里面了，基本上开箱即用

### 其他

borm的使用也比较简单

```shell
go get github.com/luanruisong/borm
```

安装之后开始初始化链接，以mysql为例（~~其他的我还没写。。~~）

```go
    cfg := &borm.MySQLConfig{
        User: "root",
        Pwd:  "123456",
        Db:   "test",
        Host: "127.0.0.1",
    }

    db, err := borm.New(cfg)
    if err != nil {
        panic(err)
    }
```

后面就可以进行直接的操作了（简单举几个例子）

```go
type TestStruct struct {
    Id      uint64 `dbw:"gt"`
    TString string
    TBool   bool
    TTime   time.Time
}

db.AutoInsert(TestStruct{
    TString: "test_insert",
    TBool:   true,
    TTime:   time.Now(),
})

db.AutoDelete(DelTestStruct{Id: 1})//大于1的删除

list := []TestStruct{}
db.SelectFrom("test_struct").All(&list)

```

### 后记

关于borm的整合，基本上就在这里了。

拖拖拉拉写了好久，这一个系列搞完，对于个人提升还是蛮大的。

尤其对于golang的reflect包使用，sql驱动包的使用，还有一些go语言设计上的认知，都有明显的提升。

写博客的过程，有利于深入挖掘自己所用过的东西，也在于总结归纳自己接触过的知识。

总体来讲，我虽然菜，没准博客写着写着就变成大神了呢~

![kFXH50](https://blog-img.luanruisong.com/blog/img/2021/05/28/kFXH50.jpg)
