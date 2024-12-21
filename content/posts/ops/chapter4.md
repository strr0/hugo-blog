---
title: "Linux 环境下消息队列的安装与使用"
date: 2023-04-25T15:00:00+08:00
categories: ["operations"]
tags: ["linux"]
draft: false
---

## 介绍

消息队列

## 1 Kafka

### 1.1 安装

#### 1.1.1 通过源码安装

##### 1.1.1.1 Java 环境

参考 [Java 环境搭建](../chapter12/#1-java-环境)

##### 1.1.1.2 Zookeeper 环境

下载 [zookeeper](https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.8.1/apache-zookeeper-3.8.1-bin.tar.gz)  
安装 zookeeper
```bash
tar -xzvf apache-zookeeper-3.8.1-bin.tar.gz -C /opt/software
cd /opt/software/apache-zookeeper-3.8.1-bin/conf
cp zoo_sample.cfg zoo.cfg
```
zk 基本配置
```sh
# /opt/software/apache-zookeeper-3.8.1-bin/conf/zoo.cfg

## zk 数据配置（可选）
dataDir=/opt/data/zookeeper
```
zk 启动与停止
```bash
/opt/software/apache-zookeeper-3.8.1-bin/bin/zkServer.sh start
/opt/software/apache-zookeeper-3.8.1-bin/bin/zkServer.sh stop
```

##### 1.1.1.3 安装

下载 [kafka](https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/3.4.0/kafka_2.12-3.4.0.tgz)  
安装 kafka
```bash
tar -xzvf kafka_2.12-3.4.0.tgz -C /opt/software
```
kafka 基本配置
```sh
# /opt/software/kafka_2.12-3.4.0/config/server.properties

## kafka 日志路径配置（可选）
log.dirs=/opt/data/kafka
## 对外地址配置
advertised.listeners=PLAINTEXT://{ip}:9092
```
kafka 启动与停止
```bash
/opt/software/kafka_2.12-3.4.0/bin/kafka-server-start.sh /opt/software/kafka_2.12-3.4.0/config/server.properties
/opt/software/kafka_2.12-3.4.0/bin/kafka-server-stop.sh
```

#### 1.1.2 通过 Docker 安装

##### 1.1.2.1 拉取镜像

```bash
docker pull bitnami/zookeeper
docker pull bitnami/kafka
```

##### 1.1.2.2 运行容器

创建网络
```bash
docker network create dev --driver bridge
```
运行 zookeeper 容器
```bash
docker run -d --name zookeeper-server \
    -p 2181:2181 \
    -e ALLOW_ANONYMOUS_LOGIN=yes \
    --network dev \
    bitnami/zookeeper:latest
```
运行 kafka 容器
```bash
docker run -d --name kafka-server \
    -p 9092:9092 \
    -e KAFKA_BROKER_ID=1 \
    -e KAFKA_CFG_LISTENERS=PLAINTEXT://:9092 \
    -e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://{ip}:9092 \
    -e KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper-server:2181 \
    -e ALLOW_PLAINTEXT_LISTENER=yes \
    --network dev \
    bitnami/kafka:latest
```

### 1.2 使用

#### 1.2.1 Spring 整合

参考 [Spring 整合 Kafka](../../dev/chapter3)

## 2 RabbitMQ

### 2.1 安装

#### 2.1.1 通过 Docker 安装

拉取镜像
```bash
docker pull rabbitmq:3-management
```
运行容器
```bash
docker run -d --hostname my-rabbit \
    --name rabbitmq-server \
    -p 5673:5672 \
    -p 15673:15672 \
    -e RABBITMQ_DEFAULT_USER=user \
    -e RABBITMQ_DEFAULT_PASS=password \
    rabbitmq:3-management
```
[管理端](http://localhost:15672)（账号密码 user/password）

### 2.2 使用

#### 2.2.1 管理端

[http://localhost:15672](http://localhost:15672)（账号密码 user/password）

#### 2.2.2 Spring 整合

参考 [Spring 整合 RabbitMQ](../../dev/chapter4)