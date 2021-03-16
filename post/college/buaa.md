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

现在发现的主要问题是，开题上报后无法修改

原因在于这个修改按钮，使用了一个

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

## 信息采集

### 平台及如何登录

平台地址 [http://47.110.138.192/](http://47.110.138.192/)

首次访问需要注册

![注册登录](https://blog-img.luanruisong.com/blog/img/20210316113710.png)

注册后登录上传即可

### 照片审核

图片审核不通过常见原因：

1. **图片要求1024*1280并且是100Kb以上的图片**

    ![错误](https://blog-img.luanruisong.com/blog/img/20210316114829.png)

2. 不要提交大头照，身体拍摄至胸部

3. 头部完整，照片清晰。

4. 不要使用翻拍的照片

5. 不要提交自拍照。

6. 不要使用后期软件将图片拉大提交，这样图片模糊依旧无法通过审核.

7. 不要在杂乱的背景前拍照.17.不要佩戴首饰。
