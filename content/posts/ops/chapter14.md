---
title: "Linux 环境下 Minio 的安装与使用"
date: 2024-08-01T10:00:00+08:00
categories: ["operations"]
tags: ["linux"]
draft: false
---

## 介绍

MinIO 是一种高性能、S3 兼容的对象存储。

## 1 安装

### 1.1 通过源码安装

#### 1.1.1 下载

```
wget https://dl.minio.org.cn/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/
```

#### 1.1.2 启动

```
MINIO_ROOT_USER=admin MINIO_ROOT_PASSWORD=password minio server /data --address ":9000" --console-address ":9001"
```

### 1.2 通过 Docker 安装

#### 1.2.1 拉取镜像

```
docker pull minio/minio:RELEASE.2023-03-24T21-41-23Z
```

#### 1.2.2 运行容器
```
docker run -d \
  --name minio-server \
  -p 9000:9000 \
  -p 9001:9001 \
  -e MINIO_ROOT_USER=admin \
  -e MINIO_ROOT_PASSWORD=password \
  -v ./minio/data:/data \
  -v ./minio/config:/root/.minio \
  minio/minio:RELEASE.2023-03-24T21-41-23Z \
  server --address ':9000' --console-address ':9001' /data
```

## 2 使用

### 2.1 管理端

[http://localhost:9001](http://localhost:9001)（账号密码 admin/password）

### 2.2 数据迁移

默认数据路径 /data

备份数据到 backup
```
mc cp --recursive /data/mybucket backup
```

### 2.3 Spring 整合

参考 [Spring 整合 Minio](../../dev/chapter14)

## 3 问题与修复

### AccessDenied

登录 minio 后台，进入桶管理（Administrator > Buckets），选择指定的桶，将 Summary 下的 Access Policy 修改为 Public。