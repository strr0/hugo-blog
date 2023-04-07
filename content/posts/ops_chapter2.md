---
title: "Docker"
date: 2023-04-06T21:00:00+08:00
draft: false
---

### 介绍
容器技术

### 安装
参考[官方文档]("https://docs.docker.com/engine/install/ubuntu/")  
①卸载旧版本
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```
②设置仓库  
更新 apt 包索引
```
sudo apt-get update
```
安装 apt 依赖包
```
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
```
③添加docker的官方gpg密钥  
~~curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg~~
由于国内网络问题这一步会失败, 更换清华镜像
```
curl -fsSL http://mirror.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
④设置稳定版仓库
```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] http://mirror.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
⑤安装docker
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
### docker基础操作
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
docker stop {镜像名:版本号}
```