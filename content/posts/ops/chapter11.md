---
title: "PagePlug 低代码平台"
date: 2023-12-05T10:00:00+08:00
categories: ["operations"]
tags: ["java"]
draft: false
---

## 介绍

PagePlug 低代码平台

## 1 快速开始

### 1.1 环境需求

- java 11
- maven 3
- mongodb
- redis

### 1.2 环境搭建

#### 1.2.1 Redis

参考 [Redis 安装](../chapter3)

#### 1.2.2 MongoDB

参考 [MongoDB 安装](../chapter5)

### 1.3 启动

#### 1.3.1 后端启动

进入后端目录
```
cd app/server
```
创建环境变量文件
```
cp envs/dev.env.example .env
```
修改环境变量
```
.env

APPSMITH_MONGODB_URI="你的 Mongo 实例地址（可不配置副本集）"
APPSMITH_REDIS_URL="你的 Redis 实例地址"
```
构建 java 服务
```
mvn clean compile
bash ./build.sh -DskipTests
```
启动 java 服务
```
bash ./scripts/start-dev-server.sh
```

> bash 命令可以使用 git bash，rsync 命令需[下载](http://mysoft.6h5.cn/win/rsync.64.zip)客户端，并解压到 {git安装路径}/usr/bin 下

#### 1.3.2 前端启动

环境变量配置
```
cp .env.example .env
```
进入前端路径
```
cd app/client
```
代理配置（可使用 nginx 代替）
```
yarn add http-proxy-middleware
```
创建 setupProxy.js
```
src/setupProxy.js

const { createProxyMiddleware } = require('http-proxy-middleware');
module.exports = function (app) {
  app.use(createProxyMiddleware('/api', {
    target: 'http://localhost:8080',
    changeOrigin: true
  }))
  app.use(createProxyMiddleware('/oauth2', {
    target: 'http://localhost:8080',
    changeOrigin: true
  }))
  app.use(createProxyMiddleware('/login', {
    target: 'http://localhost:8080',
    changeOrigin: true
  }))
};
```
启动前端服务
```
yarn
yarn start-win
```
启动成功后访问 http://localhost:3000 既可（初次使用需要创建用户）