---
title: "k-means初探"
date: 2021-03-18T12:25:28+08:00
description: "k-means 算法原理及golang实现"
categories:
    - "Development"
tags:
    - "Golang"
    - "Homework"
keywords:
    - "北航"
    - "北京航空航天大学"
    - "buaa"
    - "kmeans"
    - "k-means"
---


### 序

为了提升自我，督促自己学习，我们今天来聊一聊k-means算法

（~~绝对不是为了应付成考的作业~~）

![B数](https://blog-img.luanruisong.com/blog/img/20210318114337.jpg)

我们今天所有的内容都围绕着一个k-means的题来展开

先上题目

![截图](https://blog-img.luanruisong.com/blog/img/20210318114447.png)

网课截图，各位看官请见谅

再次为了偷懒，把答案放在前面，方便我提交作业的时候直接上链接

![棒棒](https://blog-img.luanruisong.com/blog/img/20210318114732.jpg)

### 解题过程

#### 参数

|最大循环| 最大偏移距离|k| 样本| 初始质心|  
| ----| -----| ----|---|--|---|
100 | 1 |2 |{1 1} {2 1} {1 2} {2 2} {4 3} {5 3} {4 4} {5 4} |{1 1} {1 2}|

#### 过程

|轮次| 1|
|--|--|
|初始质心| [{1 1} {1 2}] |
|根据初始质心划分簇| 1:[{1 1} {2 1}] 2:[{1 2} {2 2} {4 3} {5 3} {4 4} {5 4}] |
|重新计算质心| [{1.5 1} {3.5 3}] |
|质心最大偏移| 2.7|

|轮次| 2|
|--|--|
|初始质心| [{1.5 1} {3.5 3}] |
|根据初始质心划分簇| 1:[{1 1} {2 1} {1 2} {2 2}] 2:[{4 3} {5 3} {4 4} {5 4}]|
|重新计算质心| [{1.5 1.5} {4.5 3.5}] |
|质心最大偏移| 1.1|

|轮次| 3|
|--|--|
|初始质心| [{1.5 1.5} {4.5 3.5}] |
|根据初始质心划分簇| 1:[{1 1} {2 1} {1 2} {2 2}] 2:[{4 3} {5 3} {4 4} {5 4}] |
|重新计算质心| [{1.5 1.5} {4.5 3.5}] |
|质心最大偏移|0.0 |

#### 答案

|簇编号| 1 |
|--|--|
|质心坐标| x:1.5,y:1.5 |
|分组样本| [{1 1} {2 1} {1 2} {2 2}] |

|簇编号| 2 |
|--|--|
|质心坐标| x:4.5,y:3.5 |
|分组样本| [{4 3} {5 3} {4 4} {5 4}] |

### k-means实现思路

咱们先来说说啥叫质心，其实就是分簇数据，分完了以后的中心

简单来讲 质心的x,y坐标就是分簇数据x,y坐标的平均值

比如轮次1的分簇为

1:[{1 1} {2 1}]

2:[{1 2} {2 2} {4 3} {5 3} {4 4} {5 4}]

那么1号的质心，x = (1+2)/2,y = (1+1)/2

也就是 {1.5,1}

同理，2号的质心 x = (1+2+4+5+4+5)/6,y = (2+2+3+3+4+4)/6

也就是 {3.5,3}

以此类推，进行循环

每次循环都包含以下几部

- 计算样本与中心的距离
- 根据样本与中心距离最小值把样本分位k个簇
- 分簇之后重新计算新的质心
- 计算质心最大偏移值（如果最大偏移值小于给定条件，结束循环）
- 覆盖原本的初始质心等待下次循环使用
- 循环此处超出给定条件，结束循环

### golang代码

首先为了方便计算，我们创建一个Point结构体用于存储坐标点,再用Res结构体来保存我们k-means的计算结果

```go
type (
    Point struct {
        X float64 `json:"x"`
        Y float64 `json:"y"`
    }

    Res struct {
        Center Point   `json:"center"`
        Group  []Point `json:"group"`
    }
)

func (p Point) Fmt() string {
    return fmt.Sprintf("x:%.1f,y:%.1f", p.X, p.Y)
}

func NewPoint(x, y float64) Point {
    return Point{
        X: x,
        Y: y,
    }
}
```

下面我们来准备工具函数

根据k 创建簇

```go
func newGroup(k int) [][]Point {
    groups := make([][]Point, k)
    for i := 0; i < k; i++ {
        groups[i] = make([]Point, 0)
    }
    return groups
}
```

计算所有样本与每个质心的距离

```go
func getDistances(center, data []Point) [][]float64 {
    distances := make([][]float64, len(center))
    for i := 0; i < len(center); i++ {
        currCenter := center[i]
        distances[i] = make([]float64, len(data))
        for j, c := range data {
            distances[i][j] = Calc(c, currCenter)
        }
    }
    return distances
}
```

根据样本与质心的距离，对样本进行分簇（选用距离最短）

```go
func splitGroup(groups [][]Point, data []Point, distances [][]float64) {
    for i, v := range data {
        minIndex := 0
        minValue := math.MaxFloat64
        for j := 0; j < len(groups); j++ {
            curr := distances[j][i]
            if curr < minValue {
                minValue = curr
                minIndex = j
            }
        }
        groups[minIndex] = append(groups[minIndex], v)
    }
}
```

接下来是我为了欧几里得距离算法准备的几个函数

主要功能就是计算两个Point之间的距离

```go
func Calc(p1, p2 Point) float64 {
    if p1.X == p2.X && p1.Y == p2.Y {
        return 0
    }
    p3 := []float64{p1.X - p2.X, p1.Y - p2.Y}
    return math.Sqrt(sum(power(p3, 2)))
}

func sum(i []float64) float64 {
    s := float64(0)
    for _, v := range i {
        s += v
    }
    return s
}

func power(i []float64, f float64) []float64 {
    r := make([]float64, len(i))
    for i, v := range i {
        r[i] = math.Pow(v, f)
    }
    return r
}
```

根据分簇计算质心的方法

```go
func getCenter(g [][]Point) []Point {
    newCenter := make([]Point, len(g))
    for i, v := range g {
        sumX, sumY := float64(0), float64(0)
        for j := range v {
            sumX += v[j].X
            sumY += v[j].Y
        }
        l := float64(len(v))
        newCenter[i] = Point{
            X: sumX / l,
            Y: sumY / l,
        }
    }
    return newCenter
}
```

以及一个挑选最大值的方法

```go
func max(a ...float64) float64 {
    f := float64(0)
    for _, v := range a {
        if v > f {
            f = v
        }
    }
    return f
}
```

以及，最终的k-means算法函数

```go

func KMeans(loopLimit, maxDistance, k int, center []Point, data []Point) []Res {
    var (
        res    = make([]Res, k) //创建最终返回结果
        groups [][]Point        //用于记录最后的分簇结果
    )
    for loop := 0; loop < loopLimit; loop++ {
        //初始化簇
        groups = newGroup(k)
        //查找与质心的距离
        distances := getDistances(center, data)
        //根据距离划分簇
        splitGroup(groups, data, distances)
        //重新计算质心
        newCenter := getCenter(groups)
        //计算质心偏离距离
        centerDistance := make([]float64, k)
        for i, v := range newCenter {
            centerDistance[i] = Calc(v, center[i])
        }
        //质心偏移条件
        x := max(centerDistance...)
        if x < float64(maxDistance) {
            break
        }
        center = newCenter //重复使用变量
    }
    for i, v := range center {
        res[i] = Res{
            Center: v,
            Group:  groups[i],
        }
    }
    return res
}
```

下面就有情我们的main函数登场

```go

func main() {
    samples := defaultData()
    centers := defaultCenters()
    loopLimit := 100
    maxDistance := 1
    k := 2
    res := km.KMeans(loopLimit, maxDistance, k, centers, samples)
    for i, v := range res {
        fmt.Println("|簇编号|", i+1, "|")
        fmt.Println("|质心坐标|", v.Center.Fmt(), "|")
        fmt.Println("|分组样本|", v.Group, "|")
    }
}
```

哦哦对了，为了代码看起来好看点，我还准备了两个创建初始质心与样本的函数

```go
//初始化样本数据
func defaultData() []km.Point {
    return []km.Point{km.NewPoint(1, 1), km.NewPoint(2, 1), km.NewPoint(1, 2), km.NewPoint(2, 2),
        km.NewPoint(4, 3), km.NewPoint(5, 3), km.NewPoint(4, 4), km.NewPoint(5, 4)}
}
//初始化质心数据
func defaultCenters() []km.Point {
    return []km.Point{km.NewPoint(1, 1), km.NewPoint(1, 2)}
}
```

至此，本次k-means 圆满成功

![帅](https://blog-img.luanruisong.com/blog/img/20210318122307.jpg)
