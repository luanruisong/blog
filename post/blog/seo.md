---
title: "SEO 两三事"
date: 2021-02-07T10:14:01+08:00
description: "SEO 相关基础知识学习"
categories:
    - "Blog"
tags:
    - "SEO"
keywords:
    - "Blog"
    - "Hugo"
    - "SEO"
    - "搜索引擎优化"
---


### 序

老样子 --> [什么是SEO？](http://baidu.luanruisong.com/?q=%E4%BB%80%E4%B9%88%E6%98%AFSEO)

相信看了上面的内容，你已经知道什么是SEO了。

国内最大的搜索引擎可以说就是百度了，那么我们从百度开始吧。

### 收录

1. 一切的开始，我们要登录 ->[百度搜索资源平台](https://ziyuan.baidu.com/)<-

2. 在->[这里](https://ziyuan.baidu.com/site/index#/)<-创建并管理我们的站点

3. 在->[这里](https://ziyuan.baidu.com/linksubmit/url)<-初步提交我们的网站

4. 在->[这里](https://ziyuan.baidu.com/linksubmit/index)<-就进入了比较重要的一部，那就是资源的提交与收录

    收录的方式分3种
    - api推送 (主动把网站链接推送给百度)

    ```shell
    curl -H 'Content-Type:text/plain' --data-binary @urls.txt "http://data.zz.baidu.com/urls?site=xxxx.com&token=$token"
    ```

    ```url
    //url.txt 具体内容
    http://www.example.com/1.html
    http://www.example.com/2.html
    ...
    ```

    - sitemap收录（网站地图，用来给搜索引擎指路）
    > 一般来讲，使用静态网页生成工具来生成的静态页面都有sitemap生成，我们只需要把sitemap的具体链接提交给百度即可
    >
    > 如 我使用hugo生成的静态页再根目录即可找到 sitemap.xml
    > 
    > 所以我们 直接像百度sitemap 提交 luanruisong.com/sitemap.xml

    - 手动提交
    > 手动提交跟第一步的api提交基本相同，区别在于是使用这个页面来提交url 而不是api

完成以上几步之后，我们就可以坐等了。

### SEO

被搜索引擎收录，仅仅是我们SEO的第一步，那么如何提高我们文章在搜索引擎当中的权重呢？

这个时候，老夫灵光一闪，医美行业相对比较重视SEO，所以一个人出现在了我的脑海
![嘉哥](http://blog-img.luanruisong.com/blog/img/20210207192207.png)

~~天地良心，这个备注我是被逼的~~

医美我嘉哥，人美话不多，专业SEO三十年

人选确定，果断像我嘉哥求助(~~疯狂微信轰炸~~)

![打电话](http://blog-img.luanruisong.com/blog/img/20210207192518.png)

![打电话](http://blog-img.luanruisong.com/blog/img/20210207192353.png)

经过我长达**15**秒的寒暄，我们开始进入主题

教练，我想做SEO。

![打篮球](http://blog-img.luanruisong.com/blog/img/20210207192725.png)

教练说，SEO的基础，在于这几个标签

```html
<head>
    <title>title</title>
    <meta name="keywords" content="keywords">
    <meta name="description" content="description">
</head>

```

字面意思：标题、关键词、描述，让我们慢慢道来

#### 关于标题

Hugo的title 是根据模板渲染出来的

```html
{{ if .IsHome -}}
<title>{{ .Site.Title }}</title>
{{- else -}}
<title>{{ .Title }} &middot; {{ .Site.Title }}</title>
{{- end }}
```

所以说，需要加入什么样的title，只需要markdown的头部描述加入相对于的内容即可

然而到了这里，就不得再提一个知识点 **长尾词** (敲黑板)

当然，醉心于软件开发的我，对于长尾词这个概念肯定是一脸懵逼的

![一脸懵逼](http://blog-img.luanruisong.com/blog/img/20210207193451.png)

这时，嘉哥及时解惑

![长尾词](http://blog-img.luanruisong.com/blog/img/20210207193719.png)

嘉哥给出的建议，title标签要使用长尾词，提高再搜索中的竞争力

内页（也就是body）里面的标题，要使用h1标签，尽量不要使用h2,h3,h4等，因为

![h1](http://blog-img.luanruisong.com/blog/img/20210207193942.png)

#### 关于关键词

网站被收录并被baiduSpider爬取之后，关键词的命中，就尤为重要。

多个关键词已**英文逗号隔开**，当我们生成一篇信的blog的时候，要尽量的多填写关键词 让我们看一下我们markdown的描述信息

```yaml
title: "SEO 两三事"
date: 2021-02-07T10:14:01+08:00
description: "SEO 相关"
categories:
    - "Blog"
tags:
    - "SEO"
keywords:
    - "Blog"
    - "Hugo"
    - "SEO"
    - "搜索引擎优化"
```

本身关键词的标签只是一个html标签，由于我们的Hugu主题没有携带keywords标签，所以我决定，自己改造一下

我选择在 themes/hyde/layouts/partials/head.html加入一个meta

```html
<meta name="keywords" content="{{if .IsHome}}{{ $.Site.Params.keywords }}{{else}}{{range $idx,$value := .Keywords }}{{if lt 0 $idx}},{{end}}{{$value}}{{end}}{{end}}" />
```

里面使用的语法是 golang html/template 包的语法，有兴趣的童鞋可以自行查看

#### 关于描述

描述相对就要简单的多，对于Hugo来讲，会吧markdown头部的description放入到生成的静态页面里

```html
<meta name="description" content="{{if .IsHome}}{{ $.Site.Params.description }}{{else}}{{.Description}}{{end}}" />
```

### 后记

一下午的时间，就这么再一边轰炸嘉哥一边跟Hugo模板斗智斗勇之中过去了。

![人间疾苦](http://blog-img.luanruisong.com/blog/img/20210207200247.png)

就在我准备继续轰炸，这时嘉哥表示

![溜了](http://blog-img.luanruisong.com/blog/img/20210207200016.png)

哦，我这该死的，无处安放的，溢出屏幕的魅力啊~

![可爱](http://blog-img.luanruisong.com/blog/img/20210207200132.png)
