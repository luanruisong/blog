---
title: "WeakAuras 不完全指北"
date: 2022-09-12T10:14:01+08:00
description: "绰号“wa不求人”是怎样练成的"
categories:
    - "WeakAuras"
tags:
    - "WeakAuras"
    - "WA"
keywords:
    - "WeakAuras"
    - "WA"
    - "插件"
    - "技能监控"
---


### 序


最近嘛。。大家懂得

![](https://blog-img.luanruisong.com/blog/img/2022202209121628107.png)

但是随着版本的更新，带来的是大批量的插件和宏的失效。

最近还总碰到群里的小伙伴来问“XX监控没啦，我的dps打不上去啦~”

![](https://blog-img.luanruisong.com/blog/img/2022202209121632897.png)

为了堵住他们嘴，不给他们机会为菜找借口，这期准备做一期WA的基础教程。


### 开始

老样子 --> [什么是WeakAuras？](http://baidu.luanruisong.com/?q=%E4%BB%80%E4%B9%88%E6%98%AFWeakAuras)


其次就是 --> [WeakAuras 官方文档地址](https://github.com/WeakAuras/WeakAuras2/wiki)

还有就是 --> [WeakAuras 下载地址](https://github.com/WeakAuras/WeakAuras2/releases)

如果下载3.x版本请注意
 - classic -> 经典怀旧
 - bcc -> tbc

是的我知道，上面基本是没人看的。

![](https://blog-img.luanruisong.com/blog/img/2022202209121635752.png)

总体来讲，WA就是一个可以让玩家自助开发一些特别定制化插件的插件

![](https://blog-img.luanruisong.com/blog/img/2022202209121636242.png)

但是对于传统的魔兽插件而言，wa的过程已经简化了很多。

比如，我们只要安装好了WA，就只需要做两个操作

“XXX 给我发一下WA ” 或者 “WA 字符串发给我”

![](https://blog-img.luanruisong.com/blog/img/2022202209121638404.gif)

但是有的时候自己还是有一些奇思妙想的，这时候就只能自己动手丰衣足食了。

### 创建

游戏内输入指令/wa 开启wa的主界面

![](https://blog-img.luanruisong.com/blog/img/2022202209121644986.png)

请忽略我这些杂乱无章的wa 有很多都是历史遗留产物。

拿我的惩戒骑举例，新版本加入了一个被动效果，叫做 ***“战争艺术”*** ，触发战争艺术会使你下一个 ***“圣光闪现”*** 或者 ***“驱邪术”*** 顺发。

![](https://blog-img.luanruisong.com/blog/img/2022202209121646063.png)

那么我们在输出的时候要监控到触发了战争艺术，才去打驱邪术，或者来一发圣闪。

说完理由，那么我们点击 ***新建 -> 图标*** 再输入一个名字，那么一个新的wa就诞生了

![](https://blog-img.luanruisong.com/blog/img/2022202209121649473.png)


这里可以看到，我创建了一个叫做 ***test1*** 的wa，并且停留在了他的图示选项卡上

![](https://blog-img.luanruisong.com/blog/img/2022202209121652659.png)

选项卡有很多，这里就不全部介绍了，只介绍我们做这个简单监控时需要的

先说图示，因为我们创建的是一个 ***图标*** 所以创建成功后，他会在屏幕中间出现一个这个玩意

![](https://blog-img.luanruisong.com/blog/img/2022202209121655029.png)

在你当前选中这个wa时，这个图标可以直接拖动位置，退出wa编辑的时候，位置也就锁定了

另外在右侧 ***图示*** 选项卡下面，可以修改他大小，位置，锚点等信息。

### 设置

下面就是wa基本上逃不开的部分 ***触发***

![](https://blog-img.luanruisong.com/blog/img/2022202209121657445.png)

看一下这里的设置，基本上对于增益效果的监控，除了名称部分，设置都是一样的，大概意思就是说当我身上有了 ***战争艺术*** 这个buff的时候，这个wa开始工作

继续往下滑，可以看到一个当前wa显示的设置

![](https://blog-img.luanruisong.com/blog/img/2022202209121659299.png)

意思就是，当我设置的这个触发器触发的时候，当前的 ***图标WA*** 就显示

另外，左侧的logo有了变化，这是wa自动截取你触发器内监控的技能图标

到这里，这个wa就已经初步的做完了，让我们来看看效果

![](https://blog-img.luanruisong.com/blog/img/2022202209121702998.png)

### 丰富

可以看到，当右上角buff栏 触发了战争艺术的时候，我们之前设置的图标也跟着亮了起来

但是由于没有设置位置和大小，挺老大个玩意直接挡在了屏幕中间，全特么给我挡住了

![](https://blog-img.luanruisong.com/blog/img/2022202209121705150.png)

所以我们这里调整了一下大小和位置

![](https://blog-img.luanruisong.com/blog/img/2022202209121706316.png)

然后可以发现，这个问号图标的右下角有个 ***"1"***

![](https://blog-img.luanruisong.com/blog/img/2022202209121707389.png)

这里可以看到，关于这个1是怎么设置的，并且关于这个 ***%s*** wa给出了详细的解释

![](https://blog-img.luanruisong.com/blog/img/2022202209121708468.png)

大小调整完了，我们现在给他加一些其他的显示（这部分都是在 ***图示*** 选项卡中的操作呦）

![](https://blog-img.luanruisong.com/blog/img/2022202209121711824.png)

比如让他根据剩余时间转起来

![](https://blog-img.luanruisong.com/blog/img/2022202209121711868.png)

光转起来不行，还得发光

![](https://blog-img.luanruisong.com/blog/img/2022202209121712371.png)

所以现在变成了这样（请忽略我触发的猫鼬，那是另一个故事了）

### 整活

又转又发光之后，还可以整点其他花活，比如动画效果。

WA内置了不少动画效果，我们先来个简单的。

![](https://blog-img.luanruisong.com/blog/img/2022202209121715267.png)

这里设置了，触发开始是弹跳出现，结束的时候淡出出去。

emmmmm 不过由于我没有动图的截屏工具，就没法演示了。

![](https://blog-img.luanruisong.com/blog/img/2022202209121725336.png)

总体来讲简单的动画效果可以直接设置，wa还内置了很丰富的动画效果。

### 结束

基本上一个关于 ***战争艺术*** 的监控基本上就写完了，这个监控在PVP和PVE的情况下都是很好用的。

至于其他职业想监控别的技能，基本上基于这个wa 直接换一下技能名字即可。

这期是简单的wa创建，后续准备再多分享一些我个人制作wa的方式以及一些不好处理的地方的制作思路。

所以。。。。。。就酱，白白！

![](https://blog-img.luanruisong.com/blog/img/2022202209121718955.png)





