---
title: "Blog 打赏插件"
date: 2021-02-04T13:14:01+08:00
description: "Hugo 集成打赏插件"
categories:
    - "Blog"
tags:
    - "Reward"
    - "Hugo"
keywords:
    - "Blog"
    - "Hugo"
    - "Reward"
    - "Wechat"
    - "Alipay"
---

## 序

是的，无情的blog机器又来了

![酷酷的](https://blog-img.luanruisong.com/blog/img/20210205131148.png)

继上次更新完[《Hugo 搭建指南》](/post/blog/hugo/) 总有一种会当凌绝顶的感jio

![膨胀](https://blog-img.luanruisong.com/blog/img/20210205131434.png)

然而在逛别人blog的过程中，对于hexo生成的那些网站花里胡哨的效果，当然还是嗤之以鼻（~~口水横流~~）的

![口水](https://blog-img.luanruisong.com/blog/img/20210205131703.gif)

然而，作为一个服务端开发工程师，实打实的知道自己不应该花费精力在一些花里胡哨的东西上！

但但但但但但是，有一个功能是万万不能没有的。

那就是 **『金主爸爸的爱』** 俗称 **『打赏』**

![暴富](https://blog-img.luanruisong.com/blog/img/20210205131622.gif)

咱先不说有用没用，最起码不能阻断别人打开我通往暴富的道路！

![暴富2](https://blog-img.luanruisong.com/blog/img/20210205131559.gif)

由此引出了今天的话题。。。

## 插件

由于我们的blog是使用Hugo搭建的，对于这部分的主题，插件功能还不太好挖掘（~~其实就是我懒得去翻那一堆英文的文档~~）

所以我本次选择了使用纯前端js的插件，这样既简单，又可以满足我需要。同时也打开了通往财富的道路。

![小机灵鬼](https://blog-img.luanruisong.com/blog/img/20210205131729.png)

说干就干，我抄起github就这么一搜，一个叫做tctip 的项目映入眼帘

传送门 --> [go to tctip](https://github.com/greedying/tctip)

粗略的看了一眼README，天呐，这就是为我量身定做的插件呢~

## 使用

想使用tctip来操作打赏，只需要引入一段js就可以了

```html
<script src="//static.tctip.com/tctip-1.0.0.min.js"></script>
<script>
new tctip({
    top: '20%',
    button: {
      id: 4,
      type: 'dashang',
    },
    list: [
      {
        type: 'alipay',
        qrImg: 'http://xxx.com/alipay.png'
      }, {
        type: 'wechat',
        qrImg: 'http://xxx.com/wechat.png'
      }
    ]
  }).init()
</script>
```

这样在你的网站右边就会有一个悬浮窗来给金主爸爸开启通道

当然，我是不介意各位看官现在扫码在线测试一下的

![打赏截图](https://blog-img.luanruisong.com/blog/img/20210205131802.png)

顺带一提，还支持多个打赏通道和9个按钮样式的支持呦~

对于这个插件，只能用4个字来形容，懒！人！福！音！

![懒人福音](https://blog-img.luanruisong.com/blog/img/20210205131821.png)

当然，对于喜欢折腾的小伙伴，送你们一个[参数说明](https://github.com/greedying/tctip#%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E)

## 后记

tctip 仓库README 推荐引入的js 版本为

```html
<script src="//static.tctip.com/tctip-1.0.4.min.js"></script>
```

根据我的测试，样式靴微有那么点问题

![一点点](https://blog-img.luanruisong.com/blog/img/20210205131838.png)

所以在本文中替换为了

```html
<script src="//static.tctip.com/tctip-1.0.0.min.js"></script>
```

实测OK的呦

不信你扫码打赏一下试试~
