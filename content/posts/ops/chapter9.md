---
title: "Ruoyi-Vue-Plus 框架"
date: 2023-10-24T10:00:00+08:00
categories: ["operations"]
tags: ["java"]
draft: false
---

## 介绍

RuoYi-Vue-Plus 是重写 RuoYi-Vue 针对分布式集群与多租户场景全方位升级（不兼容原框架）

## 1 快速开始

### 1.1 环境需求

- java 17
- mysql 5.7 8.0
- redis 5.X 6.X 7.X
- maven 3.6.3 3.8.X
- nodejs >= 14
- npm 8.X

### 1.2 初始化

#### 1.2.1 Redis 环境搭建

参考 [Redis 安装](../chapter3)

#### 1.2.2 MySql 环境搭建

参考 [MySQL 安装](../chapter12/#4-mysql-环境)

#### 1.2.3 拉取代码

```bash
git clone https://gitee.com/dromara/RuoYi-Vue-Plus.git
```

#### 1.2.4 初始化数据

依次导入 sql 数据：

- script/sql/ry_vue_5.X.sql
- script/sql/test.sql
- script/sql/powerjob.sql

### 1.3 应用启动

应用列表

- MonitorAdminApplication 为 Admin 监控服务（非必要，可参考对应文档关闭）
- PowerJobServerApplication 为任务调度中心服务（非必要，可参考对应文档关闭）
- RuoYiApplication 为主应用服务

### 1.4 前端

安装依赖
```bash
npm install --registry=https://registry.npmmirror.com
```
启动服务
```bash
npm run dev
```

## 2 部署

### 2.1 maven 打包

```bash
mvn clean package -D maven.test.skip=true -P prod
```

### 2.2 Admin 监控服务

构建 ruoyi-monitor-admin 镜像并启动
```bash
docker build -t ruoyi/ruoyi-monitor-admin:5.1.0 ruoyi-extend/ruoyi-monitor-admin
docker run -d \
    --name ruoyi-monitor-admin \
    -p 9090:9090 \
    ruoyi/ruoyi-monitor-admin:5.1.0
```

### 2.3 任务调度中心服务

构建 ruoyi-powerjob-server 镜像并启动
```bash
docker build -t ruoyi/ruoyi-powerjob-server:5.1.0 ruoyi-extend/ruoyi-powerjob-server
docker run -d \
    --name ruoyi-powerjob-server \
    -p 7700:7700 \
    -p 10086:10086 \
    -p 10010:10010 \
    ruoyi/ruoyi-powerjob-server:5.1.0
```

### 2.4 主应用服务

构建 ruoyi-admin 镜像并启动
```bash
docker build -t ruoyi/ruoyi-server:5.1.0 ruoyi-admin
docker run -d \
    --name ruoyi-server \
    -p 8080:8080 \
    ruoyi/ruoyi-server:5.1.0
```
