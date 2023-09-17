---
title: "Linux环境下Docker的安装与使用"
date: 2023-04-06T21:00:00+08:00
draft: false
---

### 介绍
容器技术

### 1 安装
#### 参考[官方文档](https://docs.docker.com/engine/install/ubuntu/)  
#### 1.1 卸载旧版本
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```
#### 1.2 设置仓库  
更新 apt 包索引
```bash
sudo apt-get update
```
安装 apt 依赖包
```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
```
#### 1.3 添加docker的官方gpg密钥  
~~curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg~~  
由于国内网络问题这一步会失败, 更换清华镜像
```bash
curl -fsSL http://mirror.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
#### 1.4 设置稳定版仓库
```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] http://mirror.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
#### 1.5 安装docker
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
### 2 Docker基础操作
启动docker
```
service docker start
```
停止docker
```
service docker stop
```
拉取镜像
```
docker pull {镜像名:版本号}
```
运行容器（-p端口映射 --name容器名 -d后台运行）
```
docker run -p 8080:8080 --name {容器名} -d {镜像名:版本号}
```
停止容器
```
docker stop {容器名}
```
### 3 普通用户 permission denied 问题
```
sudo gpasswd -a ${USER} docker   # 将当前用户添加到docker组
newgrp docker                    # 更新用户组
```