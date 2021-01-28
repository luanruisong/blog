---
title: "ssh 生成信任连接"
date: 2015-10-10T13:14:01+08:00
description: "ssh 生成信任连接|ssh 不打密码"
categories:
    - "Linux"
tags:
    - "Linux"
    - "SSH"
---

## 1. ssh key

- 检查SSH keys是否存在
  ```
    cd ～/.ssh
  ```
  如果有文件id_rsa.pub 或 id_dsa.pub
  则不需要生成
-  生成新的ssh key
  ```
    ssh-keygen -t rsa -C  "your_email@example.com"
  ```
## 2. 信任连接
 -  把公钥追加到目标服务器
   ```
   cat ～/.ssh/id_rsa.pub | ssh user@host -p port "cat - >> ~/.ssh/authorized_keys"
   ```
## 3. 连接
   ```
    ssh user@host -p port
   ```
