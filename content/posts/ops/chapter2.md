---
title: "Linux 环境下 Docker 的安装与使用"
date: 2024-11-24T10:00:00+08:00
categories: ["operations"]
tags: ["linux", "mysql"]
draft: false
---

## 介绍

[Docker Engine](https://docs.docker.com/engine/install/binaries/) 是一种开源容器化技术，用于构建和容器化应用程序。

## 1 环境要求

- 64 位系统
- 内核版本 3.10 及以上
- iptables 版本 1.4 及以上
- git 版本 1.7 及以上
- ps 命令（procps 或其他包提供）
- XZ Utils 版本 4.9 及以上
- cgroupfs

## 2 安装

### 2.1 获取二进制文件

```bash
wget https://download.docker.com/linux/static/stable/<architecture>/docker-<version>.tgz
```

### 2.2 安装二进制文件

解压
```bash
tar -xzvf docker-<version>.tgz
```
安装
```bash
sudo cp docker/* /usr/bin/
```

### 2.3 运行

```bash
sudo dockerd &
```

### 2.4 配置国内镜像源

修改 /etc/docker/daemon.json
```json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.1panel.live"
  ]
}
```
重启 docker
```sh
PID=`ps -ef | grep dockerd | awk '{print $2}'`
if [ ! -z "$PID" ]
then
   sudo kill -9 $PID
fi
sudo dockerd &
```

## 3 使用

### 3.1 基础操作

查看镜像列表
```bash
docker images
```
查看容器列表
```bash
docker ps -a
```
拉取镜像
```bash
docker pull <myimage>:<version>
```
删除镜像
```bash
docker rmi <myimage>
```
运行容器
```bash
docker run -dit \             # -d 后台运行 -i 交互形式 -t tty
    --name <mycontainer> \    # --name 容器名
    -p 8080:8080 \            # -p 端口映射
    <myimage>:<version>
```
修改容器默认命令
```bash
docker run -d \
    --name <mycontainer> \
    <myimage>:<version> <mycommand>
```
停止容器
```bash
docker stop <mycontainer>
```
删除容器
```bash
docker rm <mycontainer>
```
进入容器
```bash
docker exec -it <mycontainer> /bin/bash
```
查看日志
```bash
docker logs -f --tail 100 <mycontainer>
```

### 3.2 网络

#### 3.2.1 操作

查看网络列表
```bash
docker network ls
```
创建网络
```bash
docker network create <mynetwork>
```
删除网络
```bash
docker network rm <mynetwork>
```

#### 3.2.2 使用

运行容器指定连接网络
```bash
docker run -d \
    --name <mycontainer> \
    --net <mynetwork> \
    <myimage>:<version>
```
已有容器连接网络
```bash
docker network connect <mynetwork> <mycontainer>
```
> 连接至相同网络的容器可以通过 http://mycontainer:port 相互访问

### 3.3 文件磁盘

复制容器文件
```bash
docker cp <mycontainer>:/path/in/container /path/on/host
```
磁盘空间查看
```bash
docker system df
```
磁盘空间清理
```bash
docker system prune -a
```
强制清理（需先停止 docker 服务）
```bash
rm -rf /var/lib/docker
```

### 3.4 导入导出

导出镜像
```bash
docker save -o <myimage_version>.tar <myimage>:<version>
```
导入镜像
```bash
docker load -i <myimage_version>.tar
```
导出容器
```bash
docker export -o <mycontainer>.tar <mycontainer>
```
导入容器
```bash
docker import <mycontainer>.tar <myimage>:<version>
```

### 3.5 代理

使用 dockerd 命令
```bash
sudo dockerd --http-proxy=socks5://127.0.0.1:1080 --https-proxy=socks5://127.0.0.1:1080 &
```
使用 daemon.json 配置
```json
{
  "proxies": {
    "http-proxy": "http://proxy.example.com:3128",
    "https-proxy": "https://proxy.example.com:3129",
    "no-proxy": "*.test.example.com,.example.org,127.0.0.0/8"
  }
}
```

### 3.6 其他操作

获取容器 command
```bash
docker inspect --format '{{.Config.Cmd}}' <mycontainer>
```
修改镜像名称和 tag
```bash
docker tag <oldname>:<oldtag> <newname>:<newtag>
```
判断容器是否存在
```sh
ID = `docker ps -a | grep <mycontainer> | awk '{print $1}'`
if [ ! -z "$ID" ]
then
   ...
fi
```
判断镜像是否存在
```sh
ID = `docker images | grep <myimage> | awk '{print $3}'`
if [ ! -z "$ID" ]
then
  ...
fi
```

### 3.7 构建镜像

#### 3.7.1 编写 Dockerfile
```Dockerfile
FROM openjdk:17-jdk-alpine3.14

RUN mkdir -p /app/lib \
    /app/config 

WORKDIR /app

ENV SERVER_PORT=9201 LANG=C.UTF-8 LC_ALL=C.UTF-8 JAVA_OPTS=""

EXPOSE ${SERVER_PORT}

ADD lib/* lib
ADD config/* config
ADD demo.jar app.jar

ENTRYPOINT java -Djava.security.egd=file:/dev/./urandom -Dserver.port=${SERVER_PORT} \
           -jar app.jar \
           -XX:+HeapDumpOnOutOfMemoryError -Xlog:gc*,:time,tags,level -XX:+UseZGC ${JAVA_OPTS}
```

#### 3.7.2 构建

```bash
docker build -t <myimage>:<version> <dockerfilepath>
```

#### 3.7.3 运行与停止

运行
```sh
img="demo"

version=$(date "+%Y-%m-%d")

imgId=`docker images | grep $img | grep $version | awk '{print $3}'`
if [ ! -z "$imgId" ]
then
  docker rmi $imgId
fi

docker build -t $img:$version .

docker run --name $img \
    -p 8080:8080 \
    -d $img:$version
```
停止
```sh
app="demo"

appId=`docker ps -a | grep $app | awk '{print $1}'`
if [ ! -z "$appId" ]
then
  docker stop $appId
  docker rm $appId
fi
```

### 3.8 插件（docker compose）

#### 3.8.1 安装

下载及安装
```bash
curl -SL https://github.com/docker/compose/releases/download/<version>/docker-compose-linux-<architecture> -o /usr/local/bin/docker-compose
```
赋予权限
```bash
chmod +x /usr/local/bin/docker-compose
```
验证安装
```bash
docker-compose --version
```

#### 3.8.2 使用

基本配置
```yml
# docker-compose.yml

version: '3'

services:
  mysql:
    image: mysql:8.0.33
    container_name: mysql
    environment:
      # 时区上海
      TZ: Asia/Shanghai
      # root 密码
      MYSQL_ROOT_PASSWORD: root
      # 初始化数据库(后续的初始化sql会在这个库执行)
      MYSQL_DATABASE: sampledb
    ports:
      - "3306:3306"
    volumes:
      # 数据挂载
      - /docker/mysql/data/:/var/lib/mysql/
      # 配置挂载
      - /docker/mysql/conf/:/etc/mysql/conf.d/
    command:
      # 将mysql8.0默认密码策略 修改为 原先 策略 (mysql8.0对其默认策略做了更改 会导致密码无法匹配)
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
    privileged: true
    network_mode: "host"
```
运行指定配置
```bash
docker-compose -f docker-compose.yml up -d
```

## 4 其他问题

### 4.1 普通用户 permission denied 问题

创建 docker 组（若不存在）
```bash
sudo groupadd docker
```
将当前用户添加到 docker 组
```bash
sudo usermod -aG docker $USER
```
更新用户组
```bash
newgrp docker
```

> 如果问题仍然存在，尝试重启 docker 服务

### 4.2 开启远程访问

修改 docker.service
```sh
# /lib/systemd/system/docker.service

# 注释如下
# ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
# 修改为
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```
重启 docker
```bash
systemctl daemon-reload & systemctl restart docker
```