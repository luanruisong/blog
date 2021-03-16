---
title: "关于北航成考 毕设相关"
date: 2021-03-16T09:58:52+08:00
description: "开题报告修改的问题"
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
