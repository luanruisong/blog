---
title: "WeakAuras 不完全指北2"
date: 2022-09-25T10:14:01+08:00
description: "假装在SW打鸡蛋"
categories:
    - "WeakAuras"
tags:
    - "WeakAuras"
    - "WA"
keywords:
    - "WeakAuras"
    - "WA"
    - "插件"
    - "事件监控"
---

### 序

之前做了一期技能监控的插件，这期准备来搞点好玩的

大家都知道，在BT难度下滑之后，2.43版本的SW 强度高的可怕。很多团队在开荒阶段忙活一下午没有染到进度。

![](https://blog-img.luanruisong.com/blog/img/2022202209251249410.png)

所以当你的地区处于 ***太阳井高地*** 的时候 总有人发个什么1之类的，借用你的DBM，看看你进度打到哪了。

![](https://blog-img.luanruisong.com/blog/img/2022202209251259862.png)

![](https://blog-img.luanruisong.com/blog/img/2022202209251348573.png)

所以，打不过事小，大家都不过，我能打得过就很值得装这么一B了

![](https://blog-img.luanruisong.com/blog/img/2022202209251255428.png)

### 我的WA

首先这里我们新建一个 ***文字*** 的WA

![](https://blog-img.luanruisong.com/blog/img/2022202209251300997.png)

起个名字，并且清空图示文字（因为我们这里不需要他显示信息）

![](https://blog-img.luanruisong.com/blog/img/2022202209251302192.png)

名字就叫 ***SW打不过去但我就想装逼专用插件-by 墩儿老板***

这里不同的是在于触发部分，要做一个事件触发器

![](https://blog-img.luanruisong.com/blog/img/2022202209251325439.png)

可以看到，我们这里选择了触发 ***CHAT_MSG_WHISPER*** 这个事件

接下来就是事件的函数代码了 ps：魔兽内置的是lua，有兴趣的小伙伴可以自己去了解一下语法细节

```lua
function(event,...)
    if event == "CHAT_MSG_WHISPER" then
        local text,target = ...
        if (string.find(aura_env.config.keyWord,text)) then
            local msg = "<DBM> "..
            UnitName("player").." 正在与 25人 - "..
            aura_env.config.boss.."交战，（当前 "..
            aura_env.config.hel.." "..
            aura_env.config.nodead.."/"..
            aura_env.config.people..") 生存"
            SendChatMessage(msg,"WHISPER",nil,target)    
        end
    end
end
```

首先 CHAT_MSG_* 这些事件都是与聊天有关，CHAT_MSG_WHISPER 指的就是蜜语聊天

触发函数后的参数有很多，主要需要了解的就只有前三个，第一个参数是固定的event ，这里就是CHAT_MSG_WHISPER

后面的参数回根据触发事件的不同，给与的参数也会不同，咱们监听的这个参数中，第一个掉膘聊天文本，第二个代表着密语对象

```lua
local text,target = ...
```

这个是lua的语法，表示接受省略参数的前两个

这里有一个东西出现了很多次，就是 ***aura_env.config*** 这个是wa内置的配置对象，可以通过他来获取一些自定义配置。

### 自定义

打开自定义选项卡，进行自定义配置（这里是因为想做一个大家自己配置的方式，所以没有直接写死密语内容）

![](https://blog-img.luanruisong.com/blog/img/2022202209251333168.png)

进入作者模式

![](https://blog-img.luanruisong.com/blog/img/2022202209251334923.png)

可以看到，这里的选项类型，针对的是对应配置的数据结构，比如这里我们选择的 ***字符串*** ，懂变成的小伙伴应该不莫生，就是String

显示的名字 表示在用户模式时，展示的配置项名字

选项键值，表示在lua代码中获取变量的名称

提示，表示在用户模式的提示信息

默认，就不用多说了，不填的时候会有一个默认值

再来回顾一下代码

```lua
function(event,...)
    if event == "CHAT_MSG_WHISPER" then
        local text,target = ...
        if (string.find(aura_env.config.keyWord,text)) then
            local msg = "<DBM> "..
            UnitName("player").." 正在与 25人 - ".. //获取当前用户觉得名称
            aura_env.config.boss.."交战，（当前 ".. //根据配置发送跟哪个boss交战
            aura_env.config.hel.." "..             //boss当前血量
            aura_env.config.nodead.."/"..          //生还者数量
            aura_env.config.people..") 生存"       //团队总人数
            SendChatMessage(msg,"WHISPER",nil,target) //调用密语函数发送给M你的玩家
        end
    end
end
```

到这里小伙应该能发现，我们这里有个 aura_env.config.keyWord 跟别的配置使用的方式不太一样

keyWord（关键字） 这的配置，主要是因为，你不能所有密语都直接回复你在与某某boss交战，所以当有一些比较简短的，没有内容的密语，我们才进行回复，比如M你个“1”、“。”之类的

这里主要针对的就是某些人 （~~痞子~~）

这里也使用了lua的字符串查找函数，string.find(),这里用于查找密语内容是否被自定义配置包含。

比如，我的配置是【1234567890,.，。】 如果有人单独M我一个1，这样的字符，就会回复他我正在与XXboss交战

### 载入

最后，这个插件为了提升作假质量，决定需要只有在sw地区时才加载，设置方式是这样的

![](https://blog-img.luanruisong.com/blog/img/2022202209251346331.png)

335表示太阳之井高地

### 结束

一切搞定，大功告成，展示一下成果

![](https://blog-img.luanruisong.com/blog/img/2022202209251347512.png)

有兴趣的小伙伴，可以自己做来玩玩，不限于SW哦~

![](https://blog-img.luanruisong.com/blog/img/2022202209251349693.png)



