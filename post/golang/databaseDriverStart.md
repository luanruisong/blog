---
title: "新坑！！！ 基于sql包的数据库驱动，启动（flag）篇"
date: 2021-02-09T13:14:01+08:00
description: "Golang 基于database/sql 包封装一个自用的数据库驱动"
categories:
    - "Development"
tags:
    - "Golang"
    - "Reflact"
    - "DataBaseDriver"
    - "sql"
keywords:
    - "Golang"
    - "Go"
    - "Reflact"
    - "sqlDriver"
    - "数据库驱动"
    - "sql"
---


### 序

临近春节，工作该完结的完结，新工作的开展也基本推到了年后。

由于疫情的原因，也决定就地过节，所以最后的这两天，如何快乐的摸鱼就成了重中之重！

![摸鱼](http://blog-img.luanruisong.com/blog/img/20210209162230.png)

无所事事，却又不甘心真的摸鱼，想做一个对社会有贡献的人，所以还是决定，开一个坑强迫自己学习吧。

![学习](http://blog-img.luanruisong.com/blog/img/20210209162358.png)

新坑以学习为最终目的，我们把目光瞄向了golang的数据库驱动，也可以理解为 database/sql包的二开

其实在几年前也做过类似的事情，但是如今看来惨不忍睹，无论是代码质量、设计思路，属实都感觉8太行

![8太行](http://blog-img.luanruisong.com/blog/img/20210209162736.png)

当然，如果有想翻我黑历史的同学也欢迎，传送门 --> [github](https://github.com/luanruisong/lql)

### 主流与非主流

我们先上几个主流的数据库驱动瞅瞅，毕竟咱们是处于学习和练习的目的，高星框架的大腿还是要抱紧！

![抱大腿](http://blog-img.luanruisong.com/blog/img/20210209163004.png)

|框架|支持数据库|star| doc |
| --- | --- | --- | --- |
|xorm| mysql、mymysql、postgres、tidb、sqlite、mssql、oracle| ![star](https://img.shields.io/github/stars/go-xorm/xorm?style=for-the-badge&logo=appveyor)| [xorm docs](https://xorm.io/)
|gorm|mysql、postgre、sqlite、sqlserver| ![star](https://img.shields.io/github/stars/go-gorm/gorm?style=for-the-badge&logo=appveyor)|[gorm docs](https://gorm.io/)|
|upper/db|PostgreSQL, MySQL, SQLite, MSSQL, QL and MongoDB|![star](https://img.shields.io/github/stars/upper/db?style=for-the-badge&logo=appveyor)|[upper/db docs](https://upper.io/db.v3)|

至于我们写的，那毕竟是妥妥的非主流框架了

不过不要气馁，非主流打败主流，也不是不可能的

![gdg](http://blog-img.luanruisong.com/blog/img/20210209163716.png)

### 设计

从整体看来，我想把一个数据库驱动包分位3个部分

1. sql生成器
    > 根据Map/Struct/Native 等方式生成sql

2. 数据迭代器
    > 将sql.Result进行迭代，封装后进行 指针赋值

3. 数据库特性及扩展
    > 这部分相对就比较零散了，比如Tx等功能的支持，sql方言的处理，常用sql的工具封装，等等

### 挖坑

不过分自负，也不妄自菲薄，抱着学习的目的，坑已挖好

![挖坑](http://blog-img.luanruisong.com/blog/img/20210209164818.png)

至于什么时候填嘛。。。

很快。。。 很快。。。

![溜](http://blog-img.luanruisong.com/blog/img/20210209164503.png)
