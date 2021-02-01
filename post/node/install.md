---
title: "linux 安装 nodejs"
date: 2015-12-21T13:13:36+08:00
description: "linux 指定gcc版本编译安装nodejs"
categories:
    - "Linux"
tags:
    - "Linux"
    - "nodejs"
---

## 1.安装gcc 4.8+

## 2.指定GCC编译版本

```shell
    CXX=/x/gcc-4.9.3/bin/g++ ./configure --prefix=/x/node
```

## 3 编译

```shell
    make -j8(启用多核编译)
```

## 4 安装

```shell
    make install
```

## 5 加入环境变量

```shell
    vim .bashrc  //添加—>export PATH=/x/node/bin:$PATH
```
