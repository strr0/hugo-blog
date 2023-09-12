---
title: "Linux环境下Redis的安装与使用"
date: 2023-04-25T15:00:00+08:00
draft: false
---

### 介绍
缓存

### Redis安装
如果使用docker容器，需安装lsb-release
```
sudo apt install lsb-release
```
更新apt仓库和下载redis
```
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt-get update
sudo apt-get install redis
```
redis基础配置
```
/etc/redis/redis.conf
## 允许远程访问
# bind 127.0.0.1 -::1
## 密码配置
requirepass password
```
redis启动与停止
```
service redis-server start
service redis-server stop
```