---
title: "Footman是怎样练成的"
date: 2023-12-09T12:14:01+08:00
description: "咱们来聊聊Lock Free Ring Buffer"
categories:
    - "Development"
tags:
    - "Golang"
    - "RingBuffer"
    - "LockFree"
keywords:
    - "Golang"
    - "Go"
    - "RingBuffer"
    - "LockFree"
    - "内存kafka"
---

## 序

啦啦啦啦啦啦,我是写bug的小行家

![](https://blog-img.luanruisong.com/blog/img/2022/202312091411145.gif)

距离上次水文,又过去了半年咯~~

作为打工人来讲,天天忙忙碌碌一回头,却不知自己都干了点啥....

![](https://blog-img.luanruisong.com/blog/img/2022/202312091413719.png)

最近在忙碌的工作当中,遇到了一些有趣的问题

感觉有些价值,整理一下

下面我们就聊聊今天的主角,我叫他 [**Footman**](![](https://github.com/luanruisong/footman))

## 起因

起因说起来比较简单,最初是我们在ws服务内部,服务端对于客户端主动发起的消息推送,是基于channel 或者 阻塞信号等行为实现的.

这时候对于推送的发起方,就需要轮训连接聚丙,然后挨个push

所以性能上不太好保证(尤其是一些地方用到了lock)

![](https://blog-img.luanruisong.com/blog/img/2022/202312091418624.png)

这个时候,俺**斌哥**提出了一个想法,能不能让客户端的write协程直接去读呢?

此时那可谓是一拍即合

![](https://blog-img.luanruisong.com/blog/img/2022/202312091420184.png)

但是在最初实现的过程中,我数据结构参考了LRU 使用了map+单向链表进行了实现

而且使用了一个读写锁

此时,**斌哥**直接提出来第一版修改意见

![](https://blog-img.luanruisong.com/blog/img/2022/202312091421617.png)

经过讨论,首先从数据结构来讲,Ring Buffer,其次在实现细节来讲,希望摒弃掉lock的操作

![](https://blog-img.luanruisong.com/blog/img/2022/202312091423681.png)

## 干货时间

首先,我们实现的目的,参考的kafka的逻辑

在这里就存在了Topic的角色(RingBuffer的主体)

且针对与内部进行了一些多Topic订阅的处理

下面挨个的介绍一下内部用到的组件

![](https://blog-img.luanruisong.com/blog/img/2022/202312091433754.png)

### Topic

先来看topic结构体以及相关函数

```go
type (
	Topic struct {
		name  *string
		limit *int
		seq   *uint64
		data  []*message
	}
)

func (t *Topic) init() {
	t.seq = ptr.Uint64(0)
	if t.limit == nil || *t.limit == 0 {
		t.limit = ptr.Int(10000)
	}
	t.data = make([]*message, *t.limit)
}

func (t *Topic) Limit(i int) {
	t.limit = ptr.Int(i)
}

func (t *Topic) Offset() uint64 {
	return atomic.LoadUint64(t.seq)
}

func (t *Topic) idx(offset uint64) int {
	return int(offset) % *t.limit
}

func (t *Topic) Find(offset uint64) ([]*message, error) {
	seq := atomic.LoadUint64(t.seq)
	switch {
	case offset > seq:
		return nil, ErrOutOfRange
	case offset == seq:
		return nil, ErrNoData
	default:
		ret := make([]*message, 0)
		// 套圈问题
		if ul := uint64(*t.limit); seq > ul {
			if off := seq - ul; off > offset {
				offset = off
			}
		}
		for offset = offset + 1; offset <= seq; offset++ {
			idx := t.idx(offset)
			node := atomic.LoadPointer((*unsafe.Pointer)(unsafe.Pointer(&t.data[idx])))
			if node != nil {
				ret = append(ret, (*message)(node))
			}
		}
		return ret, nil
	}
}

func (t *Topic) Append(tg any) {
	msg := NewMessage(t.name, tg)
	t.AppendMessage(msg)
}

func (t *Topic) AppendMessage(msg *message) {
	seq := atomic.AddUint64(t.seq, 1)
	idx := t.idx(seq)
	msg.offset = ptr.Uint64(seq)
	atomic.StorePointer((*unsafe.Pointer)(unsafe.Pointer(&t.data[idx])), unsafe.Pointer(msg))
}
```

这里主要是实现了一个RingBuffer,且对他的message offset分配,基于offset查询等功能的支持 

并且利用atomic包进行原子操作,避免锁机制的引入

### Server

Server 主要是针对与多个topic的管理,包含创建,初始化,删除等等功能

```go

type (
	Svr struct {
		topics *sync.Map
		opts   []Option
	}
)

func NewSvr(o ...Option) *Svr {
	return &Svr{
		topics: &sync.Map{},
		opts:   o,
	}
}

func (s *Svr) t(topicName string) *Topic {
	t := &Topic{
		name: ptr.String(topicName),
	}
	for i := range s.opts {
		s.opts[i](t)
	}
	return t
}

func (s *Svr) LoadTopic(topicName string) *Topic {
	t, loaded := s.topics.LoadOrStore(topicName, s.t(topicName))
	topic := t.(*Topic)
	if !loaded {
		topic.init()
	}
	return topic
}

func (s *Svr) RemoveTopic(topicName string) {
	_, loaded := s.topics.Load(topicName)
	if loaded {
		s.topics.Delete(topicName)
	}
}

func (s *Svr) Subscribe(topic ...string) *Consumer {
	return NewConsumer(s).init().Subscribe(topic...)
}

func (s *Svr) Produce(topic string, data any) {
	s.LoadTopic(topic).Append(data)
}

func (s *Svr) ProduceMessage(msg *message) {
	s.LoadTopic(msg.Topic()).AppendMessage(msg)
}
```

### processor

processor 主要用于管理当前consumer对于某个topic的进度

processors 主要是对于一组进度管理的封装

```go
type (
    processor struct {
        offset uint64
        topic  *Topic
    }
    processors struct {
        m   *sync.Map
        svr *Svr
    }
)


func (p *processor) find() ([]*message, error) {
    msg, err := p.topic.Find(p.offset)
    if err == nil && len(msg) > 0 {
        p.offset = msg[len(msg)-1].Offset()
    }
    return msg, err
}

func (p *processors) Subscribe(topic ...string) {
    for _, v := range topic {
        if _, ok := p.m.Load(v); !ok {
            top := p.svr.LoadTopic(v)
            p.m.Store(v, &processor{
                offset: top.Offset(),
                topic:  top,
            })
        }
    }
}

func (p *processors) Find() ([]*message, error) {
    var (
        ret   = make([]*message, 0)
        limit int
        ch    = make(chan []*message)
        ech   = make(chan error)
        err   error
    )
    p.m.Range(func(key, value any) bool {
        limit++
        go func(pp *processor) {
            info, ce := pp.find()
            if ce != nil && !errors.Is(ce, ErrNoData) {
                ech <- ce
                return
            }
            ch <- info
        }(value.(*processor))
		return true
	})
	for i := 0; i < limit; i++ {
        select {
        case info := <-ch:
            ret = append(ret, info...)
        case e := <-ech:
            if !errors.Is(e, ErrNoData) {
                err = e
            }
        }
    }
    close(ch)
    close(ech)
    return ret, err
}
```

### Consumer

由于核心的查询功能,在processors里已经实现,所以Consumer这里只需要管理processors以及对于find函数进行一些超时管理即可

```go
type(
    Consumer struct {
        timerAfter time.Duration
        svr        *Svr
        ps         *processors
    }
)


func (c *Consumer) Timer(d time.Duration) *Consumer {
    c.timerAfter = d
    return c
}

func (c *Consumer) Subscribe(topic ...string) *Consumer {
    c.ps.Subscribe(topic...)
    return c
}

func (c *Consumer) init() *Consumer {
    c.ps = &processors{
        m:   &sync.Map{},
        svr: c.svr,
    }
    c.Timer(time.Second / 10)
    return c
}

func (c *Consumer) ReadMessage(d time.Duration) ([]*message, error) {
    timer := time.NewTimer(0)
    ctx, cancel := context.WithCancel(context.Background())
    if d > 0 {
        ctx, cancel = context.WithTimeout(ctx, d)
    } else if d == 0 {
        ctx, cancel = context.WithTimeout(ctx, time.Second/2)
    }
    defer cancel()
    for {
        select {
        case <-timer.C:
            ret, err := c.ps.Find()
            if err != nil {
                return nil, err
            }
            if len(ret) > 0 {
                return ret, nil
            }
            timer.Reset(c.timerAfter)
        case <-ctx.Done():
            return nil, ErrReadTimeout
        }
    }
}

```

## 结尾

以上就是整体的实现过程以及思路了.

个人感觉还是有一些参考价值,不过暂时还没有投入使用,欢迎大家拍砖.

打工人继续搬砖去也

![](https://blog-img.luanruisong.com/blog/img/2022/202312091447399.gif)