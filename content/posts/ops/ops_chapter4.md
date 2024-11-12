---
title: "Linux 环境下 Redis 的安装与使用"
date: 2023-04-25T15:00:00+08:00
tags: ["linux"]
draft: false
---

## 介绍  
缓存

## 1 安装  

### 1.1 下载源码  
获取指定版本源码  
```
wget https://download.redis.io/releases/redis-6.2.14.tar.gz
```

### 1.2 编译  
直接编译  
```
tar -xzvf redis-6.2.14.tar.gz
cd redis-6.2.14
make
```
编译（带 LTS 支持，需要下载 openssh）  
```
make BUILD_TLS=yes
```

### 1.3 安装  
安装到 /usr/local/bin  
```
sudo make install
```

### 1.4 问题与解决  
#### 1.4.1 fatal error: jemalloc/jemalloc.h: No such file or directory  
jemalloc 重载了 Linux 下的 ANSI C 的 malloc 和 free 函数，使用如下命令编译  
```
make MALLOC=libc
```

## 2 使用
redis 基础配置
```
/etc/redis/redis.conf
## 允许远程访问
# bind 127.0.0.1 -::1
## 密码配置
requirepass password
```
redis 启动
```
redis-server
```