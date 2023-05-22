---
title: "Golang入坑指南-包管理工具"
date: 2023-05-21T13:26:37+08:00
description: "Golang入坑指南-包管理工具  go mod 的使用与配置"
categories:
    - "GoLearning"
tags:
    - "Golang"
    - "GoLearning"
keywords:
    - "Golang"
    - "Go"
---

## 序

时光又荏苒，岁月又如梭。

距离上次写相关内容的博客已经快2年了，最近准备在有限的精力当中抽出一部分，填填坑

## go mod

golang 在1.13版本正事启用了go mod 作为官方的包管理工具，彻底统一了包管理工具群魔乱舞的场面。（有点书同文、车同轨的意思）

![](https://blog-img.luanruisong.com/blog/img/2022/202305220925660.png)

## usage

### init
初始化gomod信息

```shell
go mod init {project_name}
```

### go get

引用一个go sdk（已jsoniter为例）

```shell
go get github.com/json-iterator/go
```

### tidy

tidy (go sum文件出现了损坏，或者没有写入gomod文件等等)

```shell
 go mod tidy     
```

### download

gomod 文件存在，但是本地没有相应版本的包

```shell
go mod download
```

## 配置

以包代理配置为例

### 指令（只对当前窗口有效）

```shell
go env -w GOPROXY=https://goproxy.cn,direct #配置七牛代理
go env -w GOPRIVATE=*.xxxx.org #配置特定仓库不走代理
```

### 环境变量

```shell
export GOPROXY=https://goproxy.cn,direct #配置七牛代理
export GOPRIVATE=*.xxxx.org #配置特定仓库不走代理
```

## 结束

![](https://blog-img.luanruisong.com/blog/img/2022/202305220931607.png)