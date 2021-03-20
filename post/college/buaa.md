---
title: "关于北航成考毕业相关"
date: 2021-03-16T09:58:52+08:00
description: "北航成考毕业的路不好走，一个个灾难性的平台，叫人生死难料"
categories:
    - "Other"
tags:
    - "js"
keywords:
    - "北航"
    - "北京航空航天大学"
    - "buaa"
    - "成人教育"
    - "成考"
    - "毕业设计"
    - "毕设"
    - "开题报告"
    - "开题"
---

## 开题

### 1. 毕设开题

平台地址 [http://bhcj.webtrn.cn/](http://bhcj.webtrn.cn/)

账号：学号

密码：身份证号

登录平台，选择 毕业设计 -> 开题上报

### 2. 开题相关问题

#### 关于开题报告无法修改

现在发现的主要问题是，开题上报后无法修改

原因在于这个修改按钮，使用了一个 window.navigate

![按钮](https://blog-img.luanruisong.com/blog/img/20210316100432.png)

```html
<img src="../../images/xiugai.gif" onclick="window.navigate("regbegincourse_edit.jsp?id=xxx";)">
```

导致再点击修改按钮的时候 console会报出一个错误

![报错](https://blog-img.luanruisong.com/blog/img/20210316100729.png)

查了一下文档

![文档](https://blog-img.luanruisong.com/blog/img/20210316101233.png)

可以看见，**window.navigate**函数只支持 IE和Opera浏览器

那么解决方法有两个

1. 换浏览器操作

2. 登录后直接访问 [http://bhcj.webtrn.cn/entity/student/graduate/graduate/regbegincourse_edit.jsp](http://bhcj.webtrn.cn/entity/student/graduate/graduate/regbegincourse_edit.jsp) 同时也发现，他后面跟的id参数没啥实际意义

#### 关于开题报告提交

部分同学再提交开题报告的时候，会出现这个问题

![失败](https://blog-img.luanruisong.com/blog/img/20210320133511.png)

首次碰到这个问题，我也是一头雾水，不过根据表现形式分析，能看出来是出现了一个内部的不可控异常

再来先来分析一下需要提交的数据

![数据](https://blog-img.luanruisong.com/blog/img/20210320133659.png)

我们可以看到，由于提交的信息与学校下发的模板不一样

所以最大引起问题的原因就在 **『主要内容』** 这里

我们来看一下关于主要内容的这部分，目前可以看到的限制是 200-2000字

但是没有过多的描述

大单猜测一下，平台数据库存储的 结构为 varchar(2000)

所以有些同学提交的内容会出现一种情况

- 提交的内容，纯文字不足2000
  
- 提交的内容，纯文字接近2000

所以，再加上一些符号等，就会造成 校验可以通过，但是数据库无法存储的情况

这个时候，由于是不可控的runtime 异常，估计平台采用了粗暴的try catch来进行错误返回，导致我们无法知晓原因

原因大体上找到了，那么再来说说如何解决

重新修改内容，让内容保证大于200，但是不要接近阈值（**2000**）

提交，发现这次成功

## 信息采集

### 平台及如何登录

平台地址 [http://47.110.138.192/](http://47.110.138.192/)

首次访问需要注册

![注册登录](https://blog-img.luanruisong.com/blog/img/20210316113710.png)

注册后登录上传即可

### 照片

照片上传要求：

1. 上传照片只能是png,gif,jpg格式照片

2. 照片规格：蓝色背景，头部完整清晰，身体拍摄至胸部。

3. 尺寸：~~1280*1024~~，文件不小于100KB(**1024*1280**)

图片审核不通过常见原因：

1. **图片要求1024*1280并且是100Kb以上的图片**

    ![错误](https://blog-img.luanruisong.com/blog/img/20210316114829.png)

2. 不要提交大头照，身体拍摄至胸部

3. 头部完整，照片清晰。

4. 不要使用翻拍的照片

5. 不要提交自拍照。

6. 不要使用后期软件将图片拉大提交，这样图片模糊依旧无法通过审核.

7. 不要在杂乱的背景前拍照.

8. 不要佩戴首饰。
