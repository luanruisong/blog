---
title: "Happy In Steam"
date: 2023-02-20T16:03:59+08:00
description: "一个Steam开放API的Golang SDK"
categories:
    - "Development"
tags:
    - "Golang"
keywords:
    - "Golang"
    - "Go"
    - "SSH"
    - "Steam"
---

### 序

2023 癸卯兔年来了，新的一年，这操蛋的生活静步准备从背后刀我！

![](https://blog-img.luanruisong.com/blog/img/2022/202302202028829.png)

今年有些不太一样，过年期间去了趟郑州，捡了个领导回家。

![](https://blog-img.luanruisong.com/blog/img/2022/202302202030735.png)

应领导要求，新的一年也不要忘了提升自己。

现在准备继续捡起来博客写点东西。

![](https://blog-img.luanruisong.com/blog/img/2022/202302202031286.png)

所以今天准备聊一聊 基于Steam API来封装的一套sdk。

### 准备

首先，想要使用Steam API 是需要去社区申请一个API Key的。

链接在这里 ↓↓↓

[https://steamcommunity.com/dev/appkey](https://steamcommunity.com/dev/appkey)

申请到了API Key 就可以进行后续的骚操作了。

### g-steam

g-steam 是我基于SteamAPI进行封装的一套SDK，用于快速的构建Golang的Steam应用。

仓库地址 -> [https://github.com/luanruisong/g-steam](https://github.com/luanruisong/g-steam)

#### 使用

g-steam 支持go mod模式标准安装

```shell
go get -u github.com/luanruisong/g-steam
```

引入sdk之后，使用appkey创建client

```go
    //Create client
    client := steam.NewClient("appkey")
````

设置授权登录之后302的回调地址

```go
    //path -> steam openid Login authentication address
    //callbackPath -> steam browser url to redirect to after successful authentication
    path := client.RenderTo(callbackPath)
```

创建一个API

```go
    //Create api object
    api := client.Api()
```

api这里才用了链式编程，进行对SteamAPI接口的访问.

同时，对api的服务/版本/接口 进行了拆解，使用起来只需要关注服务和版本相关就可以。

```go
    //raw return
    raw, err := api.Server("ISteamUser").//Set up service interface
    Method("GetPlayerSummaries").//Set access function
    Version("v0002").//Set version
    AddParam("steamids", "76561198421538055").//Setting parameters (If the key parameter is not set, the client's appKey will be added by default)
    Get(nil) //Initiate a request, and support the incoming structure pointer to receive parameters
    fmt.Println(raw, err) 
```

以上对于steam 的基础api调用就已经完成，但是需要根据接口文档进行相关的参数填充。

#### 扩展

查API文档是个令人头疼的问题，所以这里在原始接口封装之后，对于Steam的Server进行了扩展与整合

g-steam除了基础的包之外，也提供了几个扩展包用于直接访问Steam接口。

```go
    //Use client to create related server
    appService := isteam_app.New(client)
    iplyerSercer := iplayer_service.New(client)
    economyService := isteam_economy.New(client)
    newsService := isteam_news.New(client)
    remoteStorageService := isteam_remote_storage.New(client)
    userService := isteam_user.New(client)
    userStatsService := isteam_user_stats.New(client)
    util := isteam_webapi_util.New(client)
    //Call the server wrapper function
```

### 结束语

g-steam这个库其实写好了有一段时间了，但是一直没有整理发布出来。

在领导的鞭策下，作为应急预案写出来也是蛮好的。

游戏见证了我的过去，现在，未来已经到来。

![](https://blog-img.luanruisong.com/blog/img/2022/202302202055819.png)

羡慕么？我的！