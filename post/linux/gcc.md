---
title: "gcc 安装"
date: "2017-01-16T13:36:01+08:00"
description: "gcc 安装"
categories:
    - "Linux"
tags:
    - "Linux"
    - "gcc"
---

## GCC安装准备

### 1.安装所需要的依赖库
    ```
        yum install gmp
        yum install mpfr
        yum install mpc
        yum install mpfr-devel
        yum install gmp-devel
        yum install mpc-devel
    ```

    ps:gcc 还有两个必要的安装条件，再检测配置的时候可能检测不出来
    1.安装gcc 的机器上必须要有gcc
    2.安装gcc 时 需要先安装 gcc-c++
### 2.编译安装gcc
#### 2.1 编译配置
    ```
        ./configure --prefix=/happyface/gcc-4.9.3 --disable-multilib
    ```
#### 2.2 编译
    ```
        make -j8//(启用多核编译)
    ```
#### 2.3 安装
    ```
        make install
    ```
#### 2.4 加入系统类库
    ```
        vim /etc/ld.so.conf
    ```
#### 2.5 配置生效
    ```
        ldconfig
    ```