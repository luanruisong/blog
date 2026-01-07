---
title: "Golang入坑指南-序章"
date: 2021-07-15T17:26:37+08:00
description: "Golang入坑指南-序章  Go开发环境搭建，语法基础"
categories:
    - "GoLearning"
tags:
    - "Golang"
    - "GoLearning"
keywords:
    - "Golang"
    - "Go"
---


### 序

时光荏苒，岁月如梭。

既，5月底结束的orm框架的blog之后，已经鸽了一个多月了。

![NKfsLh](https://blog-img.luanruisong.com/blog/img/2021/07/15/NKfsLh.jpg)

曾经**小王**同学跟我说过：

你写的这些玩意吧，厉害归厉害（反正他也不知道），但是对于新手或者未入门的朋友们并不友好。

所以我们这次开的新坑应该是以拉新为主。

主要面对的，是对Golang不太了解，甚至对编程都不太了解的小伙伴们。

![XH55xZ](https://blog-img.luanruisong.com/blog/img/2021/07/15/XH55xZ.jpg)

### Golang环境安装

#### 下载Golang

首先，先把官网贴出来，有条件的小伙伴可以去自己选择版本下载

==> [go语言官网](https://golang.org/)

选择对应的版本，安装（win/mac/linux）

#### 环境变量

一般来讲，安装成功之后，需要配置环境变量

拿mac举例，需要在~/.bash_profile里面加上一行

```shell
export PATH=$PATH:{/usr/local}/go/bin 
```

***注：大括号部分要根据你具体安装到的目录来定**

刷新文件

```shell
source ~/.bash_profile
```

win小伙伴直接用此电脑->右键->设置->高级设置->环境变量设置即可

打开终端，输入

```shell
go version
```

查看安装环境变量配置情况

![UM9HqY](https://blog-img.luanruisong.com/blog/img/2021/07/15/UM9HqY.png)

### Golang开发环境安装

目前主流的开发环境有两个

- [Goland](https://www.jetbrains.com/go/)
- [VSCode](https://code.visualstudio.com/)

***注：goland收费，并且他家是维权狂魔**

当然你可以下载free版本，能白嫖30天算30天

![AfxYNB](https://blog-img.luanruisong.com/blog/img/2021/07/15/AfxYNB.jpg)

### 哈咯沃德

#### 现在我们开始我们的第一行go代码

先创建一个工作文件夹

然后再创建文件 main.go

加上我们的代码

```go
package main

import "fmt"

func main(){
    fmt.Println("Hello word")
}
```

#### 运行

再终端输入指令,就可以看到我们的运行结果了

```shell
go run main.go
```

![sdw1BG](https://blog-img.luanruisong.com/blog/img/2021/07/15/sdw1BG.png)

#### 解释

下面我们以main.go为例，简单的讲解一下设计到的一些go语言用法

麻雀虽小，五脏俱全

![ogAvtZ](https://blog-img.luanruisong.com/blog/img/2021/07/15/ogAvtZ.jpg)

别看我们这个文件只有短短的5行代码，坦白讲，他基本上就代表了go世界的缩影

首先

```go
package main

import "fmt"
```

在每个go文件的头部，都需要这两部分的定义和声明

- package 表示当前文件属于哪个包（go的世界里，包为最小单位）
- import 表示当前文件引用了哪些其他的包（fmt等这些属于官方提供的包，直接用就可以，不需要三方包管理）

简单来说，这两行说明白了，我是谁，我要用谁

再来看下面的部分

```go
func main(){
    fmt.Println("Hello word")
}
```

简单来说go语言的函数定义是这么个结构

```go
func 函数名(参数名 参数类型) 返回值类型 {
    //代码代码代码
    //我不重要我不重要
    return 返回值
}
```

- func关键字表示声明一个函数
- 函数名（参数名 参数类型）返回值类型 定义了这个函数的出入参数
- return 关键字表示要返回的嗯具体值
- "双引号" 在go语言中标识一个字符串
- fmt.Println(xxxx)表示我要再控制台打印

是的，我知道是我啰嗦了，对于大部分有开发经验的同学来讲，这里其实看一下使用方法即可。

但是，我要水字数咯~

![M84rSF](https://blog-img.luanruisong.com/blog/img/2021/07/15/M84rSF.jpg)

其次就是

1. main函数为go语言的入口函数

2. 每一个可执行的go文件，需要声明再main包下（就是文件头部的 package main）

3. 当使用go run 指令来运行你的go文件时，他执行的就是你go文件下的main函数

### 完结

至此，我们安装并写完了第一个go程序

并对go语言的基础语法，做了一丢丢解释（真的是一丢丢）

以下为内容整理

- 包声明
- 引入包声明
- 函数定义
- 其他包函数调用
- 字符串声明

好的，同学们，下课！

![anyv1w](https://blog-img.luanruisong.com/blog/img/2021/07/15/anyv1w.jpg)
