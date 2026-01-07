---
title: "Golang Reflect"
date: 2020-04-07T21:14:01+08:00
description: "Golang Reflect 包对于未知参数的处理"
categories:
    - "Development"
tags:
    - "Golang"
    - "Reflect"
keywords:
    - "Golang"
    - "Go"
    - "Reflect"
    - "反射"
    - "解析"
---

## 1 前言

- reflect包基本基于两个基础类型

```go
    reflect.Value //值
    reflect.Type  //类型
```

- value/type 的获取方法

```go
    p := struct{
        Name string
        Value string
    }{
        Name:"n",
        Value:"v",
    }
    t := reflect.TypeOf(p)   
    v := reflect.ValueOf(p)
```

- value/type 可以使用kind函数获得具体类型，大致类型如下

```go
    Bool
    Int
    Int8
    Int16
    Int32
    Int64
    Uint
    Uint8
    Uint16
    Uint32
    Uint64
    Uintptr
    Float32
    Float64
    Complex64
    Complex128
    Array       //数组
    Chan        //channel
    Func        //函数
    Interface   //
    Map         //map
    Ptr         //指针
    Slice       //切片
    String      //字符串
    Struct      //结构体
    UnsafePointer
```

## 2 未知参数的解析

- 如果kind为基础数据类型可以直接使用.(int)这种方式强转处理
- 如果kind为ptr（指针） 可以通过 value.Elem() 转换具体值
- 如果kind为struct

```go
    for i := 0; i < t.NumField(); i++ {
        f := t.Field(i)  //获取结构体属性类型，可以用来获取tag等
        vf := v.Field(i) //获取结构体属性具体值
    }
```

- 如果kind为map

```go
    iter := v.MapRange()//获取map迭代器
    for iter.Next() {
        k := iter.Key()     //K/V都是value对象，可以进一步解析
        iv := iter.Value()
        //TODO k/v
    }
```

- 如果kind为map数组或切片

```go
    if v.kind() == reflect.Slice || v.kind() == reflect.Array  {
        for idx := 0 ;idx < iv.Len() ; idx ++ {
            currValue := iv.Index(idx)//获取切片或数组中的某一元素
            //TODO currvalue
        }
    }
```

## 3 未知参数的创建

- 获取一个未知变量的类型

```go
    //以下两种获取方式都可以
    //这里忽略int等基础类型，主要针对于结构体，结构体指针等
    t := reflect.TypeOf(p)
    t:=reflect.ValueOf(p).Type()
```

- 创建这个变量

```go
     sptr := reflect.New(t)//此时得到结构体指针
     structValue := sptr.Elem() //此时获得结构体变量
```

- 创建一个结构体切片

```go
    elemType := reflect.ValueOf(p).Type() //获取切片单个元素的类型
    sliceType := reflect.SliceOf(elemType) //获取结构体切片类型
    sliceValue := reflect.MakeSlice(sliceType, 0, 0) //创建切片
    slicePtr := reflect.New(sliceValue.Type()) //创建切片指针
```

- 创建一个结构体map

```go
    //获取k/v类型
    kType := reflect.TypeOf(key)
    vType := reflect.TypeOf(value)
    //获取要创建的map Type
    mType := reflect.MapOf(kType,vType)
    //创建这个map
    mapValue := reflect.MakeMap(mType)//make
    results := reflect.New(mapValue.Type())//指针
```

## 4 未知参数的赋值

- 解析未知结构体并赋值

```go
    //假设我们获取了一个struct的类型t
    for i := 0; i < t.NumField(); i++ {
        vf := v.Field(i) //获取结构体属性具体值
        vf.Set(reflect.ValueOf(1)) //给这个属性赋值
        //更多个性化赋值
        // vf.SetString("1") 
        // vf.SetInt(1)
        // vf.SetBool(true)
        // ...
    }
```
