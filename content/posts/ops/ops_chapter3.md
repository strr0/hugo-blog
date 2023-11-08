---
title: "Linux 环境下 Kafka 的安装与使用"
date: 2023-04-24T15:00:00+08:00
tags: ["linux"]
draft: false
---

## 介绍
消息队列

## 1 前置准备
### 1.1 java 环境搭建  
### 1.2 zookeeper 环境搭建
下载 [zookeeper](https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.8.1/apache-zookeeper-3.8.1-bin.tar.gz)  
安装 zookeeper
```
tar -xzvf apache-zookeeper-3.8.1-bin.tar.gz -C /opt/software
cd /opt/software/apache-zookeeper-3.8.1-bin/conf
cp zoo_sample.cfg zoo.cfg
```
zk 基本配置
```
/opt/software/apache-zookeeper-3.8.1-bin/conf/zoo.cfg
## zk 数据配置（可选）
dataDir=/opt/data/zookeeper
```
zk 启动与停止
```
/opt/software/apache-zookeeper-3.8.1-bin/bin/zkServer.sh start
/opt/software/apache-zookeeper-3.8.1-bin/bin/zkServer.sh stop
```

## 2 Kafka 安装
下载 [kafka](https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/3.4.0/kafka_2.12-3.4.0.tgz)  
安装 kafka
```
tar -xzvf kafka_2.12-3.4.0.tgz -C /opt/software
```
kafka 基本配置
```
/opt/software/kafka_2.12-3.4.0/config/server.properties
## kafka 日志路径配置（可选）
log.dirs=/opt/data/kafka
## 对外地址配置
advertised.listeners=PLAINTEXT://{ip}:9092
```
kafka 启动与停止
```
/opt/software/kafka_2.12-3.4.0/bin/kafka-server-start.sh /opt/software/kafka_2.12-3.4.0/config/server.properties
/opt/software/kafka_2.12-3.4.0/bin/kafka-server-stop.sh
```