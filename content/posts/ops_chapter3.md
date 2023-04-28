---
title: "常用框架组件安装"
date: 2023-04-24T15:00:00+08:00
draft: false
---

### 介绍
消息队列、缓存

### Kafka安装
①java环境搭建
②zookeeper环境搭建
1. 下载[zookeeper](https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.8.1/apache-zookeeper-3.8.1-bin.tar.gz)
2. 安装zookeeper
```
tar -xzvf apache-zookeeper-3.8.1-bin.tar.gz -C /opt/software
cd /opt/software/apache-zookeeper-3.8.1-bin/conf
cp zoo_sample.cfg zoo.cfg
```
3. 基本配置（/opt/software/apache-zookeeper-3.8.1-bin/conf/zoo.cfg）
```
zoo.cfg

## zk数据配置（可选）
dataDir=/opt/data/zookeeper
```
4. zk启动与停止
```
/opt/software/apache-zookeeper-3.8.1-bin/bin/zkServer.sh start
/opt/software/apache-zookeeper-3.8.1-bin/bin/zkServer.sh stop
```
③kafka环境搭建
1. 下载[kafka](https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/3.4.0/kafka_2.12-3.4.0.tgz)
2. 安装kafka
```
tar -xzvf kafka_2.12-3.4.0.tgz -C /opt/software
```
3. 基本配置（/opt/software/kafka_2.12-3.4.0/config/server.properties）
```
server.properties

## kafka日志路径配置（可选）
log.dirs=/opt/data/kafka
## 对外地址配置
advertised.listeners=PLAINTEXT://{ip}:9092
```
4. kafka启动与停止
```
/opt/software/kafka_2.12-3.4.0/bin/kafka-server-start.sh /opt/software/kafka_2.12-3.4.0/config/server.properties
/opt/software/kafka_2.12-3.4.0/bin/kafka-server-stop.sh
```

### Redis安装
1. 如果使用docker容器，需安装lsb-release
```
sudo apt install lsb-release
```
2. 更新apt仓库和下载redis
```
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis
```
3. 基础配置（/etc/redis/redis.conf）
```
redis.conf

## 允许远程访问
# bind 127.0.0.1 -::1
## 密码配置
requirepass password
```
4. redis启动与停止
```
service redis-server start
service redis-server stop
```
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