---
title: "Ruoyi-Vue-Plus 框架"
date: 2023-10-24T10:00:00+08:00
tags: ["java"]
draft: false
---

### 介绍
RuoYi-Vue-Plus 是重写 RuoYi-Vue 针对分布式集群与多租户场景全方位升级（不兼容原框架）

### 1 部署

#### 1.1 环境需求
- java17+
- mysql
- redis
- minio
- docker

#### 1.2 Ruoyi-Plus 后端部署
拉取代码
```
git clone https://gitee.com/dromara/RuoYi-Vue-Plus.git
```
依次导入 sql 数据 script/sql/ry_vue_5.X.sql script/sql/test.sql script/sql/powerjob.sql  
maven 打包
```
mvn clean package -D maven.test.skip=true -P prod
```
构建 ruoyi-monitor-admin 镜像并启动
```
docker build -t ruoyi/ruoyi-monitor-admin:5.1.0 ruoyi-extend/ruoyi-monitor-admin
docker run -d --name ruoyi-monitor-admin -p 19090:9090 ruoyi/ruoyi-monitor-admin:5.1.0
```
构建 ruoyi-powerjob-server 镜像并启动
```
docker build -t ruoyi/ruoyi-powerjob-server:5.1.0 ruoyi-extend/ruoyi-powerjob-server
docker run -d --name ruoyi-powerjob-server -p 17700:7700 -p 10086:10086 -p 10010:10010 ruoyi/ruoyi-powerjob-server:5.1.0
```
构建 ruoyi-admin 镜像并启动
```
docker build -t ruoyi/ruoyi-server:5.1.0 ruoyi-admin
docker run -d --name ruoyi-server -p 18080:8080 ruoyi/ruoyi-server:5.1.0
```

#### 1.3 Ruoyi-Plus 前端部署
拉取代码
```
git clone https://gitee.com/JavaLionLi/plus-ui.git
```
安装依赖
```
npm install --registry=https://registry.npmmirror.com
```
启动服务
```
npm run dev
```