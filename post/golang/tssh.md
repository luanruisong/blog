---
title: "轻量级ssh管理工具--tssh"
date: 2021-04-01T16:03:59+08:00
description: "使用go语言实现一个轻量级ssh管理工具"
categories:
    - "Development"
tags:
    - "Golang"
keywords:
    - "Golang"
    - "Go"
    - "SSH"
---

### 序

消失了一阵子，主要是因为搬砖的工作实在太忙，没有时间发挥我的天性，写点沙雕blog

![sorry](https://blog-img.luanruisong.com/blog/img/20210401160630.png)

在最近的工作当中，碰到了一些问题。

主要是因为我习惯使用的 命令行ssh工具，不能存储密码导致的

以前呢，我都是使用shell自己维护一个密码文件夹，然后调用excpet脚本来进行password的识别以及密码输入。

大概是这个样子

```excpet
spawn ssh $user@$host
expect {
    "*password:" { send "$passwd\r" }
    "yes/no" { send "yes\r";exp_continue }
}
interact
```

但是最近越看越不顺眼，尤其是拙劣的文本读取，过滤等操作

![没眼看](https://blog-img.luanruisong.com/blog/img/20210401161353.png)

所以准备拿go直接实现一个我想要的样子。

### 给自己立需求

给自己确定的需求我准分两大部分

1. 基础需求
   1. 实现ssh登录
   2. 实现多种方式登录
2. 支撑需求
   1. 提供命令行参数维护需要链接的服务器

基于以上两大部分，我再给细化成几个小点

- 基础需求
  - 实现ssh登录
  - 实现tab等命令提示
  - 实现密码登录与密钥登录两种模式
- 支撑需求
  - 添加一个服务
  - 修改一个服务
  - 删除一个服务
  - 查看服务列表

### 开搞

需求已经确定了，那我们现在开搞

![搞事情](https://blog-img.luanruisong.com/blog/img/20210401161829.png)

### 支撑需求

支撑需求比较简单，基本上都是一些io的处理，所以我们先搞定这部分

先定义一个结构体用于处理我们保存的服务器信息

```go
type SSHConfig struct {
    Name   string
    Ip     string
    User   string
    Pwd    string
    SshKey string
    Port   int
    SaveAt string
}
```

里面有几个比较重要的函数

```go
func (s *SSHConfig) SaveToPath(path string) error {
    b, e := json.MarshalIndent(s, "", " ")
    if e != nil {
        return e
    }
    err := ioutil.WriteFile(path, b, os.ModePerm)
    if err == nil {
        fmt.Println("save", s.Name, " success")
    }
    return err
}

func GetFromPath(path string) (s *SSHConfig, e error) {
    var b []byte
    b, e = ioutil.ReadFile(path)
    if e != nil {
        return
    }
    s = &SSHConfig{}
    e = json.Unmarshal(b, s)
    return
}
```

这部分很简单，基本上就是把我们拿到的服务器信息，使用json的形式保存再磁盘的一个路径上。

我们再定义一个环境变量

```go
const EnvName = "TSSH_HOME"
```

这样其实我们的程保存的信息基本上都会保存在环境变量的**TSSH_HOME**里面

加上一个环境变量检测函数

```go
func DefaultCheck() error {
    configPath = os.Getenv(EnvName)
    if len(configPath) == 0 {
        return fmt.Errorf("env '%s' not found,please set a dir in env", EnvName)
    }

    if !fileExists(configPath) {
        return os.MkdirAll(configPath, os.ModePerm)
    }
    return nil
}
```

下面是存储包对外开放的函数接口

```go

func ConfigExists(name string) bool {
    return fileExists(path.Join(configPath, name))
}

func Get(name string) (*SSHConfig, error) {
    finalPath := path.Join(configPath, name)
    if !fileExists(finalPath) {
        return nil, fmt.Errorf("config %s not exists", name)
    }
    return GetFromPath(finalPath)
}

func Del(name string) error {
    finalPath := path.Join(configPath, name)
    if !fileExists(finalPath) {
        return fmt.Errorf("config %s not exists", name)
    }
    err := os.Remove(finalPath)
    if err == nil {
        fmt.Println("delete", name, "success")
    }
    return err
}

func Set(cfg *SSHConfig) error {
    finalPath := path.Join(configPath, cfg.Name)
    if fileExists(finalPath) {
        _ = os.Remove(finalPath)
    }
    return cfg.SaveToPath(finalPath)
}

func List() ([]SSHConfig, error) {
    dir, err := ioutil.ReadDir(configPath)
    if err != nil {
        return nil, err
    }
    res := make([]SSHConfig, 0)
    for _, v := range dir {
        cfg := SSHConfig{}
        b, e := ioutil.ReadFile(path.Join(configPath, v.Name()))
        if e != nil {
            return nil, e
        }
        if e = json.Unmarshal(b, &cfg); err == nil {
            res = append(res, cfg)
        } else {
            return nil, e
        }
    }
    return res, nil
}

func Env() {
    fmt.Println("env", EnvName, "=", configPath)
}
```

至此，支撑工作基本上就完成了

### 基础需求

简单的做完了，剩下的就是这一块最难啃的骨头了

![啃骨头](https://blog-img.luanruisong.com/blog/img/20210401162518.png)

这里我们采用了golang的 x/crypto/ssh 包来进行ssh链接

根据auth方式的不通创建config的代码

```go

func PwdCfg(user, pwd string) *ssh.ClientConfig {
    return &ssh.ClientConfig{
        User: user,
        Auth: []ssh.AuthMethod{ssh.Password(pwd)},
        HostKeyCallback: func(hostname string, remote net.Addr, key ssh.PublicKey) error {
            return nil
        },
        Timeout: 10 * time.Second,
    }
}

func PkCfg(user, pkPath string) (*ssh.ClientConfig, error) {
    pemBytes, err := ioutil.ReadFile(pkPath)
    if err != nil {
        return nil, fmt.Errorf("Reading private key file failed %v", err)
    }

    signer, err := ssh.ParsePrivateKey(pemBytes)
    if err != nil {
        return nil, fmt.Errorf("Parsing plain private key failed %v", err)
    }
    return &ssh.ClientConfig{
        User: user,
        Auth: []ssh.AuthMethod{ssh.PublicKeys(signer)},
        HostKeyCallback: func(hostname string, remote net.Addr, key ssh.PublicKey) error {
            return nil
        },
        Timeout: 10 * time.Second,
    }, nil
}
```

根据IP，Config等信息，创建链接的函数

```go
func Connect(ip string, port int, cfg *ssh.ClientConfig) (*ssh.Client, error) {
    addr := fmt.Sprintf("%s:%d", ip, port)
    sshClient, err := ssh.Dial("tcp", addr, cfg)
    if err != nil {
        return nil, err
    }
    return sshClient, nil
}
```

事情到了这里，感觉一切都顺利的有些过分，然而我眉头轻轻一皱，发现事情并没有那么简单

![没有那么简单](https://blog-img.luanruisong.com/blog/img/20210401162847.png)

因为在链接ssh之后，我们采用了ssh/terminal包来获取一些当前窗口的信息

导致在创建的session中出现了两个问题

首先 ssh/terminal 获取fd 以及宽高的时候，再windows下并不兼容，轻则报错，重则panic

![panic](https://blog-img.luanruisong.com/blog/img/20210330183152.png)

其次就是采用了VT100指令集，这里就算成功拿到宽高，VT100的tab等命令回写，再windows里看起就像是乱码

所以以我本人的一己之力，成功的让go的跨平台成为了一个笑话

![可笑](https://blog-img.luanruisong.com/blog/img/20210401163647.png)

拉来帮我测试windows的小伙伴，也用一种神奇的目光看着我。

所以我找了个时间，再windows环境下集中的测试了一下，发现这个鬼包就是不支持windows啊

![哭](https://blog-img.luanruisong.com/blog/img/20210401163457.png)

就在此时，我再stackoverflow上面发现了一个神奇的包 [containerd/console](https://github.com/containerd/console)

这个包完全实现了我需要的目的，通过使用编译选项的方式解决了我跨平台时终端大小获取的问题

同时也成功的在windows下采用了他该用的指令集（虽然我不知道是啥，但不是VT100）

所以，补全了我代码拼图的最后一块

```go
func RunTerminal(c *ssh.Client, in io.Reader, stdOut, stdErr io.Writer) error {
    session, err := c.NewSession()
    if err != nil {
        return err
    }
    defer session.Close()
    session.Signal(ssh.SIGINT)
    session.Stdout = stdOut
    session.Stderr = stdErr
    session.Stdin = in
    var (
        current = console.Current()
        ws      console.WinSize
    )
    defer current.Reset()

    if err = current.SetRaw(); err != nil {
        return err
    }

    if ws, err = current.Size(); err != nil {
        return err
    }

    // Set up terminal modes
    modes := ssh.TerminalModes{
        ssh.ECHO:          1,     //打开回显
        ssh.TTY_OP_ISPEED: 14400, //输入速率 14.4kbaud
        ssh.TTY_OP_OSPEED: 14400, //输出速率 14.4kbaud
        ssh.VSTATUS:       1,
    }

    //Request pseudo terminal
    if err = session.RequestPty("xterm-256color", int(ws.Height), int(ws.Width), modes); err != nil {
        return err
    }

    if err = session.Shell(); err != nil {
        return err
    }
    return session.Wait()
}
```

至此，我们给自己定义的所有需求，都完全搞定

### 整合需求

需求都弄完了，我们现在要搞个main文件来规定一下入口及参数

这里我们是用的flag包，毕竟简单，原生支持这两点已经无需多说了

```go
if err := store.DefaultCheck(); err != nil {
    fmt.Println(err)
    return
}

var (
    a = flag.String("a", "", "add config {user@host}")
    s = flag.String("s", "", "set config {user@host}")
    d = flag.String("d", "", "del config {name}")
    c = flag.String("c", "", "connect config host {name}")
    l = flag.Bool("l", false, "config list")
    e = flag.Bool("e", false, "evn info")
    v = flag.Bool("v", false, "app version")
)

var (
    n = flag.String("n", "", "set name in (-a|-s)")
    p = flag.String("p", "", "set password in (-a|-s)")
    P = flag.Int("P", 22, "set port in (-a|-s)")
    k = flag.String("k", "", "set private_key path in (-a|-s)")
)

flag.Parse()
```

多的就不写了，有兴趣的朋友可以去[github](https://github.com/luanruisong/tssh)查看

记得帮我 star，fork，watch 三连击

数据够了咱就上homebrew

### 一键安装

写到这里，主体部分基本上已经都OK了

还剩下点边边角角，比如，如何让用户快速的安装？

常规想法，咱也上个homebrew吧，毕竟一键安装爽的很

但是，根据他们要求的格式，写好脚本，commit，发起pr之后

一个邮件让我坠入了深渊。。。。

![fuck](https://blog-img.luanruisong.com/blog/img/20210330204817.png)

好吧，star，fork，watch这种东西，还是比较随缘的，我们还是先考虑如何让用户便捷的直接安装吧。。

![放弃](https://blog-img.luanruisong.com/blog/img/20210401164618.png)

这里我们编写了一个 install.sh,并使用一个比较简单的方式来进行安装

```go
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/luanruisong/tssh/master/install.sh)"
```

我们的 install.sh里面 基本上就包含了几个部分

先定义一下我用的一些参数

```shell
tag=/usr/local/bin/tssh
cpu_brand=$(sysctl machdep.cpu |grep brand_string)
down=https://github.91chifun.workers.dev/https://github.com//luanruisong/tssh/releases/download/

```

然后讨了个巧直接使用github的api来获取最新版release

```shell
version=$(wget -qO- -t1 -T2 "https://api.github.com/repos/luanruisong/tssh/releases/latest" | grep "tag_name" | head -n 1 | awk -F ":" '{print $2}' | sed 's/\"//g;s/,//g;s/ //g')
```

然后根据当前cpu信息选择下载哪个二进制包

```shell
suffex=intel
result=$(echo $cpu_brand | grep "Apple M1")
if [[ "$result" != "" ]]
 then
     suffex=appleSilicon
fi
sudo wget -O $tag $down$version/tssh-$suffex
sudo chmod +x $tag
```

这里是直接把二进制文件放到了***/usr/local/bin/tssh***

所以需要sudo 并输入本机密码，后续的chmode +x 也是一样

就这样一键安装就完成了，喜欢的小伙伴可以去下载一个玩玩。

![结束](https://blog-img.luanruisong.com/blog/img/20210401165804.png)
