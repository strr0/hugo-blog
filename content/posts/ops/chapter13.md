---
title: "Jenkins 自动化部署"
date: 2024-07-12T15:00:00+08:00
categories: ["operations"]
tags: ["linux"]
draft: false
---

## 介绍

Jenkins 是一个开源软件项目，是基于 Java 开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件项目可以进行持续集成。

## 1 部署

### 1.1 环境要求

- 机器要求：

256 MB 内存，建议大于 512 MB

10 GB 的硬盘空间（用于 Jenkins 和 Docker 镜像）

- 需要安装以下软件：

Java 8 （JRE 或者 JDK 都可以）

Docker （导航到网站顶部的 Get Docker 链接以访问适合您平台的 Docker 下载）

### 1.2 安装及初始化

#### 1.2.1 下载

下载 [Jenkins](http://mirrors.jenkins.io/war-stable/latest/jenkins.war)  

#### 1.2.2 运行

打开终端进入到下载目录，运行命令
```
java -jar jenkins.war --httpPort=8080
```
打开浏览器进入链接 [http://localhost:8080](http://localhost:8080) 按照说明完成安装

#### 1.2.3 配置国内镜像

进入 jenkins 工作目录（如 /home/centos/.jenkins）
```
cd /home/centos/.jenkins
```
修改 hudson.model.UpdateCenter.xml
```
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json</url>
  </site>
</sites>
```
修改 updates/default.json，全文替换地址
```
:%s/www.google.com/www.baidu.com/g
:%s/https:\/\/updates.jenkins.io\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g
```

#### 1.2.4 重启

```
nohup java -jar jenkins.war --httpPort=8080 > jenkins.log &
```

### 1.3 问题与修复

无法访问 jenkins，检查端口是否开放
```
firewall-cmd --list-ports
```
如果端口未开放，运行如下命令开放
```
firewall-cmd --permanent --zone=public --add-port=8080/tcp
systemctl reload firewalld
```

## 2 使用

### 2.1 基础

#### 2.1.1 插件安装

Manage Jenkins > Plugins > Available plugins

- Maven Integration plugin
- Git plugin

#### 2.1.2 全局配置

Manage Jenkins > Tools

配置 jdk、maven、git 等

#### 2.1.3 凭证配置

Manage Jenkins > Credentials

在 Stores scoped to Jenkins 列表的 Domains 下面 (global) 点击 Add credentials

### 2.2 Maven 项目配置

#### 2.2.1 构建一个 maven 项目

如 springboot-demo

#### 2.2.2 配置源代码

在 Source Code Management 选择 Git

配置仓库地址和凭证及分支等信息

#### 2.2.3 配置 maven options

如 clean package -U -Dmaven.test.skip=true

#### 2.2.4 配置后置脚本

在 Post Steps 底下点击 Add post-build step 选择 Execute shell
```
#!/bin/bash
BUILD_ID=springboot-demo

JVM_OPTION="-Xms256m -Xmx1024m"
FILE_PATH="/home/centos/workspace/springboot-demo/springboot-demo.jar"
LOG_PATH="/home/centos/workspace/springboot-demo/springboot-demo.out"
PROFILES="prod"

PID=`ps -ef | grep springboot-demo | grep java | awk '{print $2}'`
if [ ! -z "$PID" ]
then
   kill -9 $PID
fi

rm -rf $FILE_PATH

cp /home/centos/.jenkins/workspace/springboot-demo/target/springboot-demo.jar $FILE_PATH

nohup java -jar -Dspring.profiles.active=$PROFILES $JVM_OPTION $FILE_PATH >> $LOG_PATH 2>&1 &
```

#### 2.2.5 配置后置脚本（docker）

在 Post Steps 底下点击 Add post-build step 选择 Execute shell
```
#!/bin/bash
BUILD_ID=springboot-demo

containerId=`docker ps -a | grep springboot-demo | awk '{print $1}'`
if [ ! -z "$containerId" ]
then
   docker stop $containerId
   docker rm $containerId
fi
imageId=`docker images | grep springboot-demo | awk '{print $3}'`
if [ ! -z "$imageId" ]
then
   docker rmi $imageId
fi

docker build -t "springboot-demo:1.0" springboot-demo

docker run -d --name springboot-demo -p 8081:8080 -v /home/centos/workspace/springboot-demo/config:/config springboot-demo:1.0
```

### 2.3 问题与修复

#### Jenkins 拉取代码报 hudson.plugins.git.GitException: Failed to fetch from xxx

配置 git 仓库地址加上 .git 后缀