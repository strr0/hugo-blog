---
title: "Linux 环境下 ByConity 的安装与使用"
date: 2023-10-11T10:00:00+08:00
categories: ["operations"]
tags: ["linux"]
draft: false
---

## 介绍

ByConity 是面向现代 IT 架构变化而设计的一款数仓系统，它采用云原生架构设计，在满足数仓用户对资源弹性扩缩容，读写分离，资源隔离，数据强一致性等多种需求的同时，并提供优异的查询，写入性能。

## 1 在 Docker 中部署

### 1.1 环境要求

- [Docker](https://docs.docker.com/engine/)
- [Docker Compose](https://docs.docker.com/compose/)

### 1.2 docker 镜像（可提前拉取镜像）

```bash
docker pull gchq/hdfs:3.3
docker pull foundationdb/foundationdb:7.1.24
docker pull byconity/byconity:0.1.0-GA
```

### 1.3 运行 ByConity

从 github 获取源代码
```bash
git clone https://github.com/ByConity/byconity-docker.git
```
使用 docker-compose 运行 ByConity
```bash
cd byconity-docker（先进入代码根路径）
docker-compose up -d
```
测试 server 是否正常工作（返回 1）
```bash
curl '127.0.0.1:8123/?query=select%20count()%20from%20system.one'
```
测试 read worker 是否正常工作（返回 1）
```bash
curl '127.0.0.1:8123/?query=select%20count()%20from%20cnch(`vw_default`,system,one)'
```
测试 write worker 是否正常工作（返回 1）
```bash
curl '127.0.0.1:8123/?query=select%20count()%20from%20cnch(`vw_write`,system,one)'
```

### 1.4 使用 ByConity 客户端

在 hdfs 创建 clickhouse 用户
```bash
./hdfs/create_users.sh
```
进入 docker 容器的 clickhouse
```bash
docker exec -it server-0 clickhouse-client --port 52145
```
执行语句测试
```bash
SHOW DATABASES;
```
复制 docker 客户端至容器外部
```bash
mkdir bin
docker cp server-0:/opt/byconity/bin/clickhouse bin
```

### 1.5 验证部署

连接到ByConity server
```bash
bin/clickhouse client --host=<your_server_host> --port=<your_server_tcp_port>  --enable_optimizer=1 --dialect_type='ANSI'
```
确保所有worker正常启动并且被识别
```sql
select * from system.workers
```
运行一些基本的SQL
```sql
CREATE DATABASE test;
USE test;
CREATE TABLE events (`id` UInt64, `s` String) ENGINE = CnchMergeTree ORDER BY id;
INSERT INTO events SELECT number, toString(number) FROM numbers(10);
SELECT * FROM events ORDER BY id;
```
确保以上运行结果是正常的

### 1.6 数据导入

导入数据到指定表
```bash
bin/clickhouse client --query "INSERT INTO example FORMAT CSV" < example.dat --format_csv_delimiter "|"
```

### 1.7 停止 ByConity

停止 ByConity 容器
```bash
docker stop daemon-manager-0 server-0 tso-0 worker-default-0 worker-write-0 fdb-0 hdfs-datanode hdfs-namenode
```
或停止所有容器
```bash
docker stop $(docker ps -aq)
```

### 1.8 配置修改及问题修复

#### 1.8.1 配置修改

容器内部的配置文件已经挂载到物理机上，修改配置在物理机上操作即可，如设置用户密码
```sh
# /<your_directory>/byconity-simple-cluster/users.yml

users:
  default:
    networks:
      ip: ::/0
    password: "new_password"  # 修改密码为new_password
    profile: default
    quota: default
```

#### 1.8.2 DB::NetException: Connection refused 问题

在 CentOS 7 上可能会遇到 clickhouse-client 连接拒绝，可尝试如下命令登录
```bash
# 进入容器内部
docker exec -it server-0 /bin/bash

# 登录 clikchouse 客户端
clickhouse-client -m -h 127.0.0.1 --port 52145 --password
```

#### 1.8.3 Too many open files 问题

遇到 random_device failed to open /dev/urandom: Too many open files 问题，可通过如下命令修改最大允许打开文件数量
```bash
ulimit -n 2048
```

#### 1.8.4 Access to file denied 问题

遇到 DB::Exception: Access to file denied: Permission denied 问题，可能是由于 hdfs 未创建 clickhouse 用户导致，尝试如下命令创建
```bash
docker exec hdfs-namenode hdfs dfs -mkdir /user
docker exec hdfs-namenode hdfs dfs -mkdir /user/clickhouse
docker exec hdfs-namenode hdfs dfs -chown clickhouse /user/clickhouse
docker exec hdfs-namenode hdfs dfs -chmod -R 775 /user/clickhouse 
```

## 2 在 K8S 中部署

### 2.1 环境要求

#### 2.1.1 K8S 环境

更新必要包
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```
添加 k8s signing key
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
添加 k8s 源
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
安装 k8s
```bash
sudo apt-get update
sudo apt-get install -y kubectl
```

#### 2.1.2 Helm 环境

添加 gpg
```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
```
安装 apt-transport-https
```bash
sudo apt-get install apt-transport-https --yes
```
添加源
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
```
安装 helm
```bash
sudo apt-get update
sudo apt-get install helm
```
添加 helm 国内源
```bash
helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
helm repo add azure https://mirror.azure.cn/kubernetes/charts
```

#### 2.1.3 Kind 环境

```bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

#### 2.1.4 Docker 环境

参考 [Docker 安装](../chapter2)

#### 2.1.5 获取 byconity-deploy 源码

```bash
git clone git@github.com:ByConity/byconity-deploy.git
cd byconity-deploy
```

### 2.2 Kind 配置 K8S 集群

docker 拉取 node 镜像
```bash
docker pull kindest/node:v1.27.3
```
创建 1 个 control-plane 和 3 个 worker 的集群
```bash
kind create cluster --config examples/kind/kind-byconity.yaml
```
验证
```bash
kubectl cluster-info
```

### 2.3 初始化 Demo

```bash
# Install with fdb CRD first
helm upgrade --install --create-namespace --namespace byconity -f ./examples/kind/values-kind.yaml byconity ./chart/byconity --set fdb.enabled=false

# Install with fdb cluster
helm upgrade --install --create-namespace --namespace byconity -f ./examples/kind/values-kind.yaml byconity ./chart/byconity
```
等待所有 pods 准备完毕（使用如下命令查看状态）
```bash
kubectl -n byconity get po
```
进入容器
```bash
kubectl -n byconity exec -it sts/byconity-server -- bash
```
进入 clickhouse 客户端
```bash
clickhouse client
```

### 2.4 从 Kubernetes 集群中删除或停止 ByConity

删除集群
```bash
helm uninstall --namespace byconity byconity
```
停止 ByConity
```bash
docker stop byconity-cluster-control-plane byconity-cluster-worker byconity-cluster-worker2 byconity-cluster-worker3
```