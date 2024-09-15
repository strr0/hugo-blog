---
title: "Linux 环境下 Docker 的安装与使用"
date: 2023-04-06T21:00:00+08:00
tags: ["linux", "mysql"]
draft: false
---

## 介绍
[Docker Engine](https://docs.docker.com/engine) 是一种开源容器化技术，用于构建和容器化应用程序。

## 1 环境要求
### 1.1 系统要求  
安装 Docker 需要如下 64 位的 Ubuntu 系统
- Ubuntu Lunar 23.04
- Ubuntu Kinetic 22.10
- Ubuntu Jammy 22.04 (LTS)
- Ubuntu Focal 20.04 (LTS)

### 1.2 卸载旧版本  
在安装 Docker 前需要卸载任何可能冲突的包
- docker.io
- docker-compose
- docker-compose-v2
- docker-doc
- podman-docker

执行如下命令卸载
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

## 2 安装
### 2.1 设置 Docker 仓库  
添加官方 GPG Key
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
添加仓库地址到 apt 源
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/ \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

### 2.2 安装 Docker
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 2.3 配置国内镜像源
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
service docker restart
```

## 3 基础操作
### 3.1 Docker
启动 docker
```
service docker start
```
停止 docker
```
service docker stop
```
拉取镜像
```
docker pull myimage:version
```
运行容器（-p 端口映射 --name 容器名 -d 后台运行）
```
docker run -p 8080:8080 --name mycontainer -d myimage:version
```
停止容器
```
docker stop mycontainer
```
进入容器
```
docker exec -it mycontainer /bin/bash
```
复制容器文件
```
docker cp mycontainer:/path/in/container /path/on/host
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
判断容器是否存在
```
if docker ps -a | grep -q mycontainer; then
  ...
fi
```
判断镜像是否存在
```
if docker images | grep -q myimage; then
  ...
fi
```

### 3.2 Docker Compose
#### 3.2.1 安装
下载及安装
```
curl -SL https://github.com/docker/compose/releases/download/v2.20.3/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```
赋予权限
```
chmod +x /usr/local/bin/docker-compose
```
验证安装
```
docker-compose --version
```
#### 3.2.2 使用
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

## 4 其他问题
### 4.1 普通用户 permission denied 问题
```
sudo gpasswd -a ${USER} docker   # 将当前用户添加到docker组
newgrp docker                    # 更新用户组
```
### 4.2 开启远程访问  
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