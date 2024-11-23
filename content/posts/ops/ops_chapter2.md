---
title: "Linux 环境下 Docker 的安装与使用"
date: 2023-04-06T21:00:00+08:00
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
```
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.1panel.live"
  ]
}
```
重启 docker  
```
PID=`ps -ef | grep dockerd | awk '{print $2}'`
if [ ! -z "$PID" ]
then
   sudo kill -9 $PID
fi
sudo dockerd &
```

## 3 使用  

### 3.1 基础操作  
拉取镜像  
```
docker pull <myimage>:<version>
```
运行容器（-p 端口映射 --name 容器名 -d 后台运行）  
```
docker run -p 8080:8080 --name <mycontainer> -d <myimage>:<version>
```
停止容器  
```
docker stop <mycontainer>
```
进入容器  
```
docker exec -it <mycontainer> /bin/bash
```

### 3.2 文件磁盘相关  
复制容器文件  
```
docker cp <mycontainer>:/path/in/container /path/on/host
```
磁盘空间查看  
```
docker system df
```
磁盘空间清理  
```
docker system prune -a
```
强制清理  
```
systemctl stop docker
rm -rf /var/lib/docker
systemctl start docker
```

### 3.3 导入导出  
导出镜像  
```
docker save -o <myimage_version>.tar <myimage>:<version>
```
导入镜像  
```
docker load -i <myimage_version>.tar
```
导出容器  
```
docker export -o <mycontainer>.tar <mycontainer>
```
导入容器  
```
docker import <mycontainer>.tar <myimage>:<version>
```

### 3.4 其他操作  
判断容器是否存在  
```
ID = `docker ps -a | grep <mycontainer> | awk '{print $1}'`
if [ ! -z "$ID" ]
then
   ...
fi
```
判断镜像是否存在  
```
ID = `docker images | grep <myimage> | awk '{print $1}'`
if [ ! -z "$ID" ]
then
  ...
fi
```

## 4 Docker Compose  

### 4.1 安装  
下载及安装
```
curl -SL https://github.com/docker/compose/releases/download/<version>/docker-compose-linux-<architecture> -o /usr/local/bin/docker-compose
```
赋予权限
```
chmod +x /usr/local/bin/docker-compose
```
验证安装
```
docker-compose --version
```

### 4.2 使用  
基本配置
```
docker-compose.yml

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
```
docker-compose -f docker-compose.yml up -d
```

## 5 其他问题  

### 5.1 普通用户 permission denied 问题  
```
sudo gpasswd -a ${USER} docker   # 将当前用户添加到docker组
newgrp docker                    # 更新用户组
```

### 5.2 开启远程访问  
修改 docker.service
```
/lib/systemd/system/docker.service

# 注释如下
# ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
# 修改为
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```
重启 docker
```
systemctl daemon-reload & systemctl restart docker
```