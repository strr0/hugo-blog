---
title: "Linux 环境下 Minio 的安装与使用"
date: 2024-08-01T10:00:00+08:00
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
MINIO_ROOT_USER=admin MINIO_ROOT_PASSWORD=password minio server /mnt/data --address ":9000" --console-address ":9001"
```

## 数据迁移  
默认数据路径 /data

备份数据到 backup
```
mc cp --recursive /data/mybucket backup
```