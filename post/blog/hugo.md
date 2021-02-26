---
title: "Hugo 搭建指南"
date: 2021-02-03T13:14:01+08:00
description: "搭建 Golang 的blog生成工具 Hugo"
categories:
    - "Blog"
tags:
    - "Hugo"
keywords:
    - "Blog"
    - "Hugo"
    - "Golang"
    - "Go"
---




## 前言

本着 总结经验 (~~方便装逼~~) 的原则，开始使用blog记录编程过程中碰到的具有代表性的问题，

最初的几篇也都是取自于原本自己的云笔记。

工作几年来，攒下的笔记不少，但是可以用作blog的寥寥无几。

本来想直接贴链接水几篇blog来的，对于我这个想法，身边的朋友都表示非常支持

![人干事](https://blog-img.luanruisong.com/blog/img/20210205132500.png)

而且发现笔记落灰严重 && 链接泛滥 && 自己都没咋看

对于这么几个问题，决定还是先从 Hugo开始吧，毕竟他是本次blog的起点

## Hugo

 1. 什么是Hugo

    P话不多说，上链接 --> [关于什么是Hugo](http://baidu.luanruisong.com/?q=%E4%BB%80%E4%B9%88%E6%98%AFHugo)

    相信看过了链接的你，一定对与Hugo 有了一定的认识，那么我们继续

    溜~!

    ![溜](https://blog-img.luanruisong.com/blog/img/20210205132516.png)

 2. Hugo的安装
    - 二进制安装 (Mac用户专享)

    ```shell
       lrs@Mr-CIH> brew install hugo
    ```

    - 二进制安装 (Linux用户专享)

      - 我懒了，去翻release吧 传送门 --> [github](https://github.com/gohugoio/hugo/releases)

    - 源码安装

      - [装git](http://baidu.luanruisong.com/?q=%E8%A3%85git)
      - [装Go（1.3+）](http://baidu.luanruisong.com/?q=%E8%A3%85go)
      - 装Hugo

    ```shell
       lrs@Mr-CIH> export GOPATH=$HOME/go
       lrs@Mr-CIH> go get -v github.com/gohugoio/hugo
    ```

 3. Hugo的使用

    - 生成站点

    ```shell
       lrs@Mr-CIH> hugo new site /path/to/site
    ```

    - 创建一个 about 页面

    ```shell
       lrs@Mr-CIH> hugo new about.md
    ```

    - 生成静态目录

    ```shell
       lrs@Mr-CIH> hugo    
    ```

 4. 其他
    - 关于md文件的头部

        每个md文件的头部都会用 toml/yaml格式标注的本篇blog的相关信息，但是官方演示代码不够全面，这边给出一个相对比较全的

    ```yaml
    ---
    title: "Hugo 搭建指南"
    date: 2021-02-03T13:14:01+08:00
    description: "搭建 Golang 的blog生成工具 Hugo"
    categories:
        - "Blog"
    tags:
        - "Golang"
        - "Hugo"
    ---
    ```

    - 内置的链接

        经过我坚持不懈的暴力破解（~~实际上就是瞎蒙~~）发现了以下两个

      - /tags/ （标签集合）
      - /categories/ （分类集合）

    - 关于 config.toml/config.yaml

        具体还是要根据主题来调整配置，我就先上几个我这边用到的通用配置好了

    ```yaml
    baseURL : "http://luanruisong.com"
    languageCode : "en-us"
    title : "anwu's blog"
    theme : "hyde"
    Menus:
        main :
            - Name : "about"
              URL : "/about/"
              Weight: 4
            - Name : "tags"
              URL : "/tags/"
              Weight: 2
            - Name : "categories"
              URL : "/categories/"
              Weight: 1
            - Name : "github"
              URL : "https://github.com/luanruisong"
              Weight: 3
            - Name : "Links"
              URL : "/links/"
              Weight: 10
    ```

## 花里胡哨的骚操作

是的，写到这里那一定是要有花里胡哨的骚操作的，不然就不是我了

![美滋滋](https://blog-img.luanruisong.com/blog/img/20210205132558.png)

我发现使用nodejs构建静态blog的童鞋们，都喜欢把整个静态页面丢到git上去

对我来说，每次要上传辣么大的文件，实在是考验我与github之间这若即若离的关系

![郁闷](https://blog-img.luanruisong.com/blog/img/20210205132616.png)

所以我下定决心，要玩点花里胡哨的东西！！！

- 首先，我们需要一个ecs 在上面安装 ***Hugo,git,nginx***
- 其次，我们把md文件的git clone 到 $HOGO_HOME/content
- 再次，我们把nginx的根目录代理到 $HOGO_HOME/public
- 再再次，编写两个脚本(或者按你心情合成一个)用来做自动更新

```shell
#!/bin/sh
cd $1 && git reset --hard && git pull
```

```shell
#!/bin/sh
cd $1 && hugo
```

- 再再再次，再github上开启一个webhook，指向到nginx
- 再再再再次，触发webhook的服务(这里我是使用go写了个小服务)，直接去调用上面那两个脚本即可
- 再再再再再次（我保证最后一个），只需要再本机clone最开始的md仓库，写markdown 然后push就好啦。
- 这样我们就完成了一个 ***push->webhook->pull->build*** 的循环

流程图奉上 （画完图觉得有点蠢，然而我不想改了）

![流程](https://blog-img.luanruisong.com/blog/img/20210205132632.png)

## 完结撒花

至此，我终于完美的水了一篇关于搭建Hugo的blog，完美~

果然，我还是那个无情的blog机器

![颤抖吧，凡人](https://blog-img.luanruisong.com/blog/img/20210205132651.png)
