---
title: "Linux 环境下 MongoDB 的安装与使用"
date: 2024-01-04T15:00:00+08:00
categories: ["operations"]
tags: ["linux"]
draft: false
---

## 介绍

非关系型数据库

## 1 安装

### 1.1 通过源码安装

#### 1.1.1 下载 [mongodb](https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1804-6.0.5.tgz)

#### 1.1.2 解压到指定路径

```
tar -xzvf mongodb-linux-x86_64-ubuntu1804-6.0.5.tgz -C /opt/software
```

#### 1.1.3 创建数据及日志目录

```
sudo mkdir -p /opt/data/mongo
sudo mkdir -p /opt/log/mongo
sudo chown `whoami` /opt/data/mongo   # 设置权限
sudo chown `whoami` /opt/log/mongo   # 设置权限
```

#### 1.1.4 基础配置

```
/etc/mongodb.conf
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

#### 1.1.5 启动及停止

```
cd /opt/software/mongodb-linux-x86_64-ubuntu1804-6.0.5/bin
./mongod -f /etc/mongodb.conf --fork
./mongod -f /etc/mongodb.conf --shutdown
```

#### 1.1.6 设置用户名密码

```
./mongod
> use admin
> db.createUser({ user: 'admin', pwd: 'password', roles: [{ role: 'root', db: 'admin' }] })
```

### 1.2 通过 Docker 安装

拉取镜像
```
docker pull mongo
```
运行容器
```
docker run -d --name mongodb-server -p 27017:27017 mongo --auth
```
设置初始化用户
```
docker exec -it mongodb-server mongosh admin
> db.createUser({ user: 'admin', pwd: 'password', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
```
创建数据库及赋予权限
```
docker exec -it mongodb-server mongosh admin
> db.auth('admin', 'password');
> use testdb;
> db.createUser({ user: 'user', pwd: 'password', roles: [ { role: "dbOwner", db: "testdb" } ] });
```

## 2 使用

### 2.1 集合操作

创建集合
```
db.createCollection("book")
```
删除集合
```
db.book.drop()
```

### 2.2 查询语句

查询文档
```
db.book.find()  ## book 为表名
```
条件查询（and）
```
db.book.find({ key1: value1, key2: value2 })
```
条件查询（or）
```
db.book.find({ $or: [{ key1: value1 }, { key2: value2 }] })
```
mongo 条件语句
|    操作    |           格式           |                 范例                |     RDBMS中的类似语句     |
| :--------: | :---------------------: | :---------------------------------: | :----------------------: |
|    等于    |    `{<key>:<value>}`     |  `db.book.find({"by":"strr"})`  | `where by = 'strr'` |
|    小于    |  `{<key>:{$lt:<value>}}` |  `db.book.find({"likes":{$lt:50}})` |    `where likes < 50`   |
|  小于或等于 | `{<key>:{$lte:<value>}}` | `db.book.find({"likes":{$lte:50}})` |   `where likes <= 50`   |
|    大于    |  `{<key>:{$gt:<value>}}` |  `db.book.find({"likes":{$gt:50}})` |    `where likes > 50`   |
|  大于或等于 | `{<key>:{$gte:<value>}}` | `db.book.find({"likes":{$gte:50}})` |   `where likes >= 50`   |
|   不等于   |  `{<key>:{$ne:<value>}}` |  `db.book.find({"likes":{$ne:50}})` |    `where likes != 50`   |

### 2.3 其他操作

插入文档
```
db.book.insert({ name: 'test book', author: 'strr' })
```
更新文档
```
db.book.update({ _id: ObjectId('1') }, { $set: { name: 'test book 1' } })
```
删除文档
```
db.book.remove({ _id: ObjectId('1') })
```

### 2.4 Spring 整合

参考 [Spring 整合 MongoDB](../../dev/chapter5)