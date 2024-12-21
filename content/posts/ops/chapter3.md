---
title: "Linux 环境下 Redis 的安装与使用"
date: 2023-04-24T15:00:00+08:00
categories: ["operations"]
tags: ["linux"]
draft: false
---

## 介绍

缓存

## 1 安装

### 1.1 通过源码安装

#### 1.1.1 获取源码

```bash
wget https://download.redis.io/releases/redis-6.2.14.tar.gz
```

#### 1.1.2 编译

直接编译
```bash
tar -xzvf redis-6.2.14.tar.gz
cd redis-6.2.14
make
```
编译（带 LTS 支持，需要下载 openssh）
```bash
make BUILD_TLS=yes
```

#### 1.1.3 安装

安装到 /usr/local/bin
```bash
sudo make install
```

#### 1.1.4 配置

```sh
# /etc/redis/redis.conf

## 允许远程访问
# bind 127.0.0.1 -::1
## 密码配置
requirepass password
```

#### 1.1.5 问题与解决

##### fatal error: jemalloc/jemalloc.h: No such file or directory

jemalloc 重载了 Linux 下的 ANSI C 的 malloc 和 free 函数，使用如下命令编译
```bash
make MALLOC=libc
```

### 1.2 通过 Docker 安装

#### 1.2.1 拉取镜像

```bash
docker pull redis
```

#### 1.2.2 运行容器

```bash
docker run -d \
    --name redis-server \
    -p 6379:6379 \
    redis --requirepass <password>
```

## 2 使用

### 2.1 基础使用

启动
```bash
redis-server
```
客户端
```bash
redis-cli
```
获取所有 key
```bash
keys *
```
设置键值对
```bash
set <key> <value>
```
获取值
```bash
get <key>
```
删除键
```bash
del <key>
```

### 2.2 Spring 整合

参考 [Spring 整合 Redis](../../dev/chapter2)