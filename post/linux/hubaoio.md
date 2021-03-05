---
title: "报哥案例收集之 自己作死自己还不记得篇"
date: 2021-03-05T11:14:01+08:00
description: "个人服务器无法ssh登陆，即便登陆也卡的无法正常执行命令"
categories:
    - "Linux"
tags:
    - "Linux"
    - "SSH"
    - "io"
keywords:
    - "Linux"
    - "SSH"
    - "io"
---


### 序

在这个风和日丽的礼拜五，一则微信消息，打破了我宁静的生活

![jquery god](https://blog-img.luanruisong.com/blog/img/20210305113745.png)

即上次我报哥问我技术问题之后，我深深地感觉到了，他碰到的问题都没辣么简单

![安排](https://blog-img.luanruisong.com/blog/img/20210305114115.jpg)

但是，被安排归被安排，问题还是要解决的，最起码我们也要给瞎出几个主意，表示咱们不是水货！

![装逼](https://blog-img.luanruisong.com/blog/img/20210305114226.jpg)

### 问题出现

首先报哥先抛出了这么个问题

![卡顿](https://blog-img.luanruisong.com/blog/img/20210305114349.png)

然后我们查看了一下他的服务器，除了cpu是个单核以外，其他的基本上跟普通ecs没差，按理说不应该卡成这个样子。

所以我们把怀疑的方向换到网络方面

![网络](https://blog-img.luanruisong.com/blog/img/20210305114740.png)

然后我们对比了一下机房线路的延时，得出结论--怀疑错了

![委屈](https://blog-img.luanruisong.com/blog/img/20210305114830.jpg)

问题方向就这么直接被掐死在了摇篮之中

![绝望](https://blog-img.luanruisong.com/blog/img/20210305115325.jpg)

就在我继续瞎BB，准备把怀疑方向扯到系统上时，报哥表示，退下！我自己来！

![嫌弃](https://blog-img.luanruisong.com/blog/img/20210305115123.gif)

### 报哥的个人秀

少了我的捣乱，报哥决定还是要从基础查起。

首先去排除一下阿里云的监控，发现磁盘IO 占用异常。查看历史7天监控，如下

![监控](https://blog-img.luanruisong.com/blog/img/20210305115454.png)

轻而易举的把问题定位到了和磁盘I/O有关（~~我是真真真真捣乱了~~）

这时准备重启系统，查看系统磁盘i/o负载情况

重启后，系统可以正常ssh，执行命令，先用iostat查看磁盘io 是否读写负载很高，结果如下

![占用](https://blog-img.luanruisong.com/blog/img/20210305115704.png)

映入眼帘的是一列红字，%iowait 90+，意思是cpu百分之90以上的时间在等待I/O。而服务器又是单核，所以这无疑就是问题所在。

好吧，那就来看下占用io最高的进程是个啥？

iotop ... ,command not found。（yum install iotop -y）

结果如下

![iotop](https://blog-img.luanruisong.com/blog/img/20210305115758.png)

可以看到，除了系统调度之外，gitlab-p...就是罪魁祸首啦，定位到问题啦。

### 后记

看到这里，我马上对报哥发起问话

![问话](https://blog-img.luanruisong.com/blog/img/20210305115855.png)

好家伙，我直接好家伙。程序员狠起来，连自己都打

![害怕](https://blog-img.luanruisong.com/blog/img/20210305120041.jpg)

至于现在，报哥苦哈哈的去把自己的gitlab进程干掉，然后嘛。。。

![装逼结束](https://blog-img.luanruisong.com/blog/img/20210305120131.jpg)
