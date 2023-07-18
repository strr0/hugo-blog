---
title: "MongoDB"
date: 2023-04-26T15:00:00+08:00
draft: false
---

### 介绍
非关系型数据库

### MongoDB安装
1. 下载[mongodb](https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1804-6.0.5.tgz)
2. 安装mongodb
```
tar -xzvf mongodb-linux-x86_64-ubuntu1804-6.0.5.tgz -C /opt/software
```
3. 创建数据及日志目录
```
sudo mkdir -p /opt/data/mongo
sudo mkdir -p /opt/log/mongo
sudo chown `whoami` /opt/data/mongo   # 设置权限
sudo chown `whoami` /opt/log/mongo   # 设置权限
```
4. mongodb配置（/etc/mongodb.conf）
```
mongodb.conf

## 数据路径
dbpath=/opt/data/mongo
## 日志路径
logpath=/opt/log/mongo/mongod.log
## 端口号
# port=27017
## 远程访问
bind_ip=0.0.0.0
## 用户验证
# auth=true
```
5. mondodb启动及停止
```
cd /opt/software/mongodb-linux-x86_64-ubuntu1804-6.0.5/bin
./mongod -f /etc/mongodb.conf --fork
./mongod -f /etc/mongodb.conf --shutdown
```
6. 设置用户名密码
```
use admin
db.createUser({
  user: 'admin',
	pwd: 'password',
	roles: [{
	  role: 'root',
		db: 'admin'
	}]
})
```