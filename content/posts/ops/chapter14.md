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

### 1.1 下载

```
wget https://dl.minio.org.cn/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/
```

### 1.2 启动

```
MINIO_ROOT_USER=admin MINIO_ROOT_PASSWORD=password minio server /data --address ":9000" --console-address ":9001"
```

## 2 使用

### 2.1 数据迁移

默认数据路径 /data

备份数据到 backup
```
mc cp --recursive /data/mybucket backup
```

### 2.2 Spring 整合

参考 [Spring 整合 Minio](../../dev/chapter14)

## 3 问题与修复

### AccessDenied

登录 minio 后台，进入桶管理（Administrator > Buckets），选择指定的桶，将 Summary 下的 Access Policy 修改为 Public。