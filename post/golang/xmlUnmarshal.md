---
title: "Xml Unmarshal"
date: 2021-01-29T17:14:01+08:00
description: "Golang Xml 反序列化相关"
categories:
    - "Development"
tags:
    - "Golang"
    - "Xml"
---


## 1. 这都不会？不是有手就行？

今日报哥给我了一个问题：xml解析用go做过吗？

![害羞](https://gitee.com/luanruisong/blog_img/raw/master//20210129171908.png)
当时我的反应是：这都不会？
![震惊](https://gitee.com/luanruisong/blog_img/raw/master//20210129172846.png)
然后就开始了我的装逼（被打脸）之旅

## 2. 让我们来康康，发生甚么事了

> 首先，大家都知道，go语言在官方sdk对于json和xml都有基础支持  
就是  encoding/xml encoding/json 包
我们惯性思维 解析xml嘛，跟json一样，一个结构体甩你脸上 直接Unmarshal就好

想象中

```go
    xmlStr := "管他娘的什么xml"
    m := XmlItems{} //对着结构写的结构体
    xml.Unmarshal([]byte(str), &m)
```

然而，快乐的时光总是短暂的，我报哥直接甩给了我一个xml

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <items>
        <item autocomplete="adfadfaaf" valid="yes">
            <title>adfaddffd</title>
            <subtitle>aaa</subtitle>
            <icon>is.png</icon>
            <text type="copy">dadafda</text>
        </item>
    </items>
```

天真的我还在想这个结构体怎么对着撸的时候，发现了几个问题

* 如果按照常规思路，子节点直接是属性，那么对于本身就是属性的地方解析有啥不同？
* 如何区分我这是个子节点的内容，还是当前节点的属性？
* 如果这个节点又具有属性，我们还需要他的具体内容，结构体该如何定义？

## 3. 这个光有手，还真不行

然而在残酷的现实中，无论我如何定义我的结构体，使用xml的tag 总有一部分解析不出来，所以不得不求助谷歌大大并发现了几个有意思的东西

* 对着xml撸结构体的时候，有几个东西要注意一下
  
```go
    xml.Name //作用域
    xml.Attr //可以直接解析属性
    `xml,xxxx` //根据不同情况附加各种写法
```

* 撸结构体的时候，最大的争议无非就是既有内容又有属性的这种，比如报哥给的xml中的 ***items[0].text,*** 
* 对于这个xml报文，对应的结构体如下
  
```go
    type XmlItems struct {
        XMLName xml.Name `xml:"items"`
        Iterm []struct {
            XMLName      xml.Name `xml:"item"`
            Autocomplete xml.Attr `xml:"autocomplete,attr"`
            Valid        xml.Attr `xml:"valid,attr"`
            Title        string   `xml:"title"`
            Subtitle     string   `xml:"subtitle"`
            Icon         string   `xml:"icon"`
            Text         struct {
                XMLName      xml.Name `xml:"text"`
                TextType xml.Attr `xml:"type,attr"`
                Content  string   `xml:",innerxml"`
            } `xml:"text"`
        } `xml:"item"`
    }
```

* 由此可见关于xml的tag逗号后面的东西  就是和解析json最主要的差别了 整理一下放到下面
  
   1. 具有标签"-"的字段会省略
   2. 具有标签"name,attr"的字段会成为该XML元素的名为name的属性
   3. 具有标签",attr"的字段会成为该XML元素的名为字段名的属性
   4. 具有标签",chardata"的字段会作为字符数据写入，而非XML元素
   5. 具有标签",innerxml"的字段会原样写入，而不会经过正常的序列化过程
   6. 具有标签",comment"的字段作为XML注释写入，而不经过正常的序列化过程，该字段内不能有"--"字符串
   7. 标签中包含"omitempty"选项的字段如果为空值会省略
   8. 空值为false、0、nil指针、nil接口、长度为0的数组、切片、映射

* 顺便在推荐个中文文档

    [https://studygolang.com/pkgdoc](https://studygolang.com/pkgdoc)

* 另外，跟我一样懒的咸鱼们，你们的福音来了
    感谢阿水，直接生成它不香么？
    [https://www.onlinetool.io/xmltogo/](https://www.onlinetool.io/xmltogo/)

## 4. 敌羞，吾去脱他衣

问题至此完全解决,我要拿着结果跟我报哥邀功去了

![得瑟](https://gitee.com/luanruisong/blog_img/raw/master//20210129175003.png)