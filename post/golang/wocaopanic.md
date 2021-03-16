---
title: "记一次有趣的panic"
date: 2021-03-16T18:10:11+08:00
description: "Golang interface{} type interface"
categories:
    - "Development"
tags:
    - "Golang"
    - "interface"
keywords:
    - "Golang"
    - "Go"
    - "interface"
---

### 序

风和日丽，是个写bug的好天气

![天气](https://blog-img.luanruisong.com/blog/img/20210316181221.jpg)

今天在写bug的过程中，碰到了一个有趣的panic

先上个示例代码,这里用到了一个结构体，一个接口

```go
type (
    Message interface {
        PrintT()
    }
    MessageImpl struct {
        t string
    }
)

func (mi *MessageImpl) PrintT() {
    fmt.Println(mi.t)
}
```

然后再加上两个函数

```go
func fmtMsg(m Message) {
    if m == nil {
        return
    }
    m.PrintT()
}

func nilMsg() *MessageImpl {
    return nil
}
```

最后加上main函数

```go
func main() {
    fmtMsg(new(MessageImpl))
    fmtMsg(nilMsg())
}
```

先来看一下运行结果

```shell
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x10a6a45]

goroutine 1 [running]:
main.(*MessageImpl).Topic(0x0, 0xc00000e018, 0xc00006af48)
        main.go:15 +0x5
main.fmtMsg(0x10ec440, 0x0)
        main.go:23 +0xa4
main.main()
        main.go:32 +0x56
```

很明显 

```go
if m == nil {
    return
}
```

这行代码识别出现了问题，那么问题来了 ，为什么我明明返回的nil但是却不能识别呢？

### 解决方案

先来说说解决方案吧，想要解决这个无法识别nil的情况，有两种方式

1. 使用反射识别,如：

    ```go
    func fmtMsg(m Message) {
        if reflect.ValueOf(m).IsNil() {
            return
        }
        m.PrintT()
    }
    ```

2. 修改返回值，如：

    ```go
    func nilMsg() Message {
        return nil
    }
    ```

使用这两种方式都可以有效的识别空值，并正常的return或者调用PrintT函数

### 原因

再来说说为什么可以用这两种方式解决

1. 使用reflect来进行空识别的原因

    >很简单，就是因为 interface{} 在golang底层的识别有异于常规的结构体或指针，他本身包含 类型，值，当赋值为空时，他的类型是有的，但是值为nil，所以单纯的==nil比较是不起作用的
    >
    >type interface{} 这个类型 与直接使用interface{} 在结构上有些区别，但是特性基本一样
    >
    >使用reflect.IsNil其实就是跳过类型直接看值，就可以解决这个问题。

2. 修改返回值为指针的原因

    > 这个就要分析我们明明没有使用interface{}但是却出现了interface{}的特性是为什么了
    >
    >首先，我们nilMsg的第一版本返回的类型是 *MessageImpl，这时，这个变量的类型就已经确定了。
    >
    >其次，在我们调用fmtMsg(m Message)这个函数的时候，可以看到他的类型为 Message接口。
    >
    >这时 由于golang的特性是复制传参，那么实际上这里做了两部操作，1.转换我们当前的变量成为Message接口，2.复制给参数使用
    >
    >再结合我们说的interface{}的特性，是不是就和带类型的nil值赋值给interface{}的情况一样了？
    >
    >所以我们在nilMsg函数return的类型直接指定为Message的时候，有个好处，再传参给fmtMsg的时候，不存在转换复制，是单纯的复制，所以nil就可以识别出来了。

### 后记

本期文字较多，表情包比较少，大家见谅。

由于大部分是语言特性的问题，至于正确性嘛。。。。

我觉得他是对的，也欢迎不同意见的小伙伴来喷我。

![搬砖](https://blog-img.luanruisong.com/blog/img/20210316184348.jpg)
