---
title: "Linux 环境下 Redis 的安装与使用"
date: 2023-04-25T15:00:00+08:00
tags: ["linux"]
draft: false
---

## 介绍
缓存

## 1 安装
### 1.1 如果使用 docker 容器，需安装 lsb-release
```
sudo apt install lsb-release
```
### 1.2 更新 apt 仓库
```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt-get update
```
### 1.3 下载 redis
```
sudo apt-get install redis
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
redis 启动与停止
```
service redis-server start
service redis-server stop
```