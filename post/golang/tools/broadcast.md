---
title: "Emmmmm....Broadcast!!"
date: 2022-08-31T16:50:40+08:00
description: "关于广播通知的一些思考与实现"
categories:
    - "Development"
tags:
    - "Golang"
    - "Channel"
    - "Signal"
    - "Broadcast"
keywords:
    - "Golang"
    - "Go"
    - "Golang"
    - "Channel"
    - "Signal"
    - "Broadcast"
    - "广播消息"
    - "广播信号"
---


### 序

时光如水，生命如歌~ 转眼间已经好久咩有更新博客~

![](https://blog-img.luanruisong.com/blog/img/2022/202208311657215.png)

首先抛开我自己的问题不谈，一定是因为工作太忙了！

![](https://blog-img.luanruisong.com/blog/img/2022/202208311656288.png)

不过，在工作的过程中，碰到了一个协程通讯的问题，就是如何在同一时间把消息通知给所有需要知道的协程呢？

提起协程，Gopher们脑子都不带转一下的，协程通讯用channel嘛，完全么的难度。

![](https://blog-img.luanruisong.com/blog/img/2022/202208311658374.png)


但是如果说，我们要给其他协程通知的内容是相同的，甚至于，可能只需要给他触发一个信号呢？

那么每个协程使用channel通知的成本就很高了，这里有个很相似的场景:

***"长链接广播"***


虽然解决这个问题之后发现，已经完全偏离了我想解决的问题，**但是** 解决的过程还是很有意思，希望记录下来。

![](https://blog-img.luanruisong.com/blog/img/2022/202208311721095.png)


### 先来点常规的

在Gopher的世界里，常规解决协程间通讯，首选channel加select

比如这样：

```go
for {
    select {
        case <- ctx.Done():
            //code
        case <- ch1:
            //code
        case <- ch2:
            //code
    }
}
```

但是我们的场景是这样

 - 有一个主协程A
 - 有几百个子协程B
 - A协程每隔时间X执行一次，进行数据预处理
 - A协程预处理数据之后，通知B协程开始进行任务
 - B协程接到信号，使用A协程预处理后的数据进行操作

由于A协程持有所有B协程task的指针，所以我们第一版的方案是这样的

![](https://blog-img.luanruisong.com/blog/img/2022/202208311740277.png)


基于这个设计，我们就出现了第二层次的问题：

 - A协程通知B协程时，需要遍历然后进行channel的写入
 - 写入channel时，需要使用协程执行，不然需要等他写完再写下一个
 - 通知的执行时间会跟着协程数线性的增长（因为是使用遍历的方式）

 那么，有没有办法解决这些问题呢？

 ![](https://blog-img.luanruisong.com/blog/img/2022/202208311745120.png)

5 minutes later 。。。

 ![](https://blog-img.luanruisong.com/blog/img/2022/202208311745844.png)

 2000 years later ...

 ![](https://blog-img.luanruisong.com/blog/img/2022/202208311746771.png)

 还是去看看别人都是怎么做的吧。。。

 ![](https://blog-img.luanruisong.com/blog/img/2022/202208311748006.png)


 ### 发现了一点不一样的

 看了很多代码，全局的信号接收一般使用channel的close来实现。

 但是像我们这种通知的场景，直接close肯定是一个不明智的选择。

 那么有没有一种可能，我们把已经close的channel重新打开捏？？？

 抱着这样的想法，我搜到了这样一篇文章

 [知乎->Golang中重新open 已经被close的chan管道](https://zhuanlan.zhihu.com/p/24722712)
 
 ![](https://blog-img.luanruisong.com/blog/img/2022/202208311842447.png)

 正在我美滋滋的沉浸在可以魔改channel的时候,发现了这么几个评论：

 ![](https://blog-img.luanruisong.com/blog/img/2022/202208311843230.png)

 顺带一提，感谢几个大佬提供的一盆冷水！让我冷静且重新思考这个问题。

![](https://blog-img.luanruisong.com/blog/img/2022/202208311844002.png)

既然觉得不要魔改，那么，就得使用一些自己的方式解决这个问题了。

### 我们来搞点事情叭

首先，我们先确定一个方向，那就是使用close channel的方式，向所有需要通知的协程发送信号。

![](https://blog-img.luanruisong.com/blog/img/2022/202208311846200.png)

但是由于channel已经被close了，那么我们需要一个可以置换channel的方法。

上代码！


```go

type (
	SignalStation struct {
		lock sync.RWMutex
		cp   chan struct{}
	}
)

func (ss *SignalStation) CurrSignal() <-chan struct{} {
	ss.lock.RLock()
	defer ss.lock.RUnlock()
	return ss.cp
}

func (ss *SignalStation) Send() {
	ss.lock.Lock()
	defer ss.lock.Unlock()
	cc := ss.cp
	ss.cp = make(chan struct{})
	close(cc)
}

func NewSignalStation() *SignalStation {
	return &SignalStation{
		lock: sync.RWMutex{},
		cp:   make(chan struct{}),
	}
}
```

这里可以看到，首先我们设置了一个读写锁，用于channel的置换以及 获取当前channel的函数。

同时，在发送信号的时候，实际上是把当前channel置换成一个全新的channel，并把老channel进行关闭。

看一下testcase

```go

func TestSignalStation(t *testing.T) {

	ss := NewSignalStation()
	for i := 0; i < 10; i++ {
		go func(idx int) {
			for {
				select {
				case <-ss.CurrSignal():
					t.Log(idx, "receive signal")
				}
			}
		}(i)
	}
	for i := 0; i < 10; i++ {
		ss.Send()
		time.Sleep(time.Second)
	}
}

```

可以看到，我们在协程中直接select中对于 **ss.CurrSignal()** 进行了读取并阻塞。

这里结合到send函数的地方，就可以发现，我们每次都可以获取到最新的，未关闭的channel

同时当我们调用send函数时，会先对channel进行置换，这样保证了新来的一轮循环中，获取的channel是新的channel

然后对于老channel，我们直接把他关闭，这样就实现了，一个channel，通知到所有协程，并可以反复使用的目的。

我可真是厉害呢！

![](https://blog-img.luanruisong.com/blog/img/2022/202208311859971.png)

### 只有通知？？ 我不满足！

目前的代码虽少，但是基本上经过了一些测试，可以完美的解决我们需要他解决的通知问题。

是时候思考一下，通知的同时，是否可以携带一些消息下去了！！！

![](https://blog-img.luanruisong.com/blog/img/2022/202208311901339.png)

众所周知，由于我们使用的一个channel 那么所有select他的协程，获取到的通讯内容，是互斥的。

所以基于我们最开始的设计思路，那么这个channel就不能作为传输数据的通道，只是传输信号。

那么我们基于信号，对他进行一层封装

```go

type (
	NoticeStation struct {
		lock   sync.RWMutex
		signal *SignalStation
		curr   interface{}
	}
)

func (ss *NoticeStation) WaitForValue() interface{} {
	<-ss.CurrSignal()
	return ss.CurrValue()
}

func (ss *NoticeStation) CurrSignal() <-chan struct{} {
	return ss.signal.CurrSignal()
}

func (ss *NoticeStation) CurrValue() interface{} {
	ss.lock.RLock()
	defer ss.lock.RUnlock()
	return ss.curr
}

func (ss *NoticeStation) OnNotice(ctx context.Context, f func(value interface{})) {
	ss.signal.OnSignal(ctx, func() {
		f(ss.CurrValue())
	})
}

func (ss *NoticeStation) OnNoticeAsync(ctx context.Context, f func(value interface{})) {
	go ss.OnNoticeAsync(ctx, f)
}

func (ss *NoticeStation) Notice(value interface{}) {
	ss.lock.Lock()
	defer ss.lock.Unlock()
	ss.curr = value
	ss.signal.Send()
}

func NewNoticeStation() *NoticeStation {
	return &NoticeStation{
		signal: NewSignalStation(),
		lock:   sync.RWMutex{},
	}
}

```

这次的代码就比较多了，主要是基于一个SignalStation进行包装，在发送信号前，对于变量Value进行一个写入（加锁）然后发送信号，并且在接收信号的第一时间进行读取（这里是读锁，众所周知，读共享，写独占嘛）


来看一下testcase

```go

func TestNoticeStation(t *testing.T) {
	ns := NewNoticeStation()

	for i := 0; i < 10; i++ {
		go func(idx int) {
			for {
				fmt.Println(idx, ns.WaitForValue())
			}
		}(i)
	}
	for i := 0; i < 10; i++ {
		ns.Notice(time.Now().Format("2006-01-02 15:04:05"))
		time.Sleep(time.Second)
	}
}

func TestOnNotice(t *testing.T) {
	ns := NewNoticeStation()

	for i := 0; i < 8000; i++ {
		go func(idx int) {
			ns.OnNotice(context.Background(), func(value interface{}) {
				fmt.Println(idx, value)
			})
		}(i)
	}
	fmt.Println("start goroutine over")
	time.Sleep(time.Second * 5)
	for i := 0; i < 10; i++ {
		ns.Notice(time.Now().Format("2006-01-02 15:04:05"))
		time.Sleep(time.Second)
	}
}

```

这里主要测试了 **WaitForValue** 与 **OnNotice** 函数的使用场景

从测试结果来看，可以很完美的对于数据传输以及信号通知。

感觉自己的代码之魂又提高了一个档次呢！

![](https://blog-img.luanruisong.com/blog/img/2022/202208311908153.png)


### 我的新库，Broadcast！！！

学而不思则罔，思而不学则殆，代码研究了不开源，不如卖白菜。

所以我吧以上的所有代码，提交到了github上进行开源，欢迎大家来使用以及拍砖。

传送门 -> [https://github.com/luanruisong/broadcast](https://github.com/luanruisong/broadcast)

对于golang的小伙伴，可以直接使用go get安装

```shell
go get -u github.com/luanruisong/broadcast
```

至此，完结撒花~

![](https://blog-img.luanruisong.com/blog/img/2022/202208311858759.gif)