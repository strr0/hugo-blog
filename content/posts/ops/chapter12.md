---
title: "基础环境配置"
date: 2024-11-11T17:00:00+08:00
categories: ["operations"]
tags: ["linux"]
draft: false
---

## 介绍

Linux 开发环境

## 1 Java 环境

### 1.1 下载

下载 [jdk](https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz)

### 1.2 安装

解压到指定路径
```bash
sudo mkdir /usr/local/java
sudo tar -xzvf jdk-17_linux-x64_bin.tar.gz -C /usr/local/java
```

### 1.3 环境变量配置

修改环境变量
```sh
# /etc/profile

JAVA_HOME=/usr/local/java/jdk-17.0.8
export PATH=$PATH:$JAVA_HOME/bin
```
应用环境变量
```bash
source /etc/profile
```

### 1.4 验证

运行命令验证
```bash
java -version
```

### 1.5 问题与修复

#### Cannot load from short array because "sun.awt.FontConfiguration.head" is null

Linux 缺失字体导致，从 jdk 1.8 的 jre/lib 复制 fontconfig.bfc 到安装的 jdk 17 的 lib 中  

## 2 Maven 环境

### 2.1 下载

下载[maven](https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz)  

### 2.2 安装

解压到指定路径
```bash
sudo mkdir /opt/maven
sudo tar -xzvf apache-maven-3.6.3-bin.tar.gz -C /opt/maven
```

### 2.3 环境变量配置

修改环境变量
```sh
# /etc/profile

M2_HOME=/opt/maven/apache-maven-3.6.3
export PATH=$PATH:$M2_HOME/bin
```
应用环境变量
```bash
source /etc/profile
```

### 2.4 验证

运行命令验证
```bash
mvn -version
```

## 3 Git 环境

### 3.1 安装

```bash
yum install git
```

### 3.2 验证

```bash
git --version
```

## 4 MySQL 环境

### 4.1 安装

#### 4.1.1 通过 Docker 安装

拉取镜像
```bash
docker pull mysql:8.0.31
```
创建容器并启动
```bash
docker run -d \
    --name mysql-server \
    -p 3306:3306 \
    -v ./mysql:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=password \
    -e MYSQL_DATABASE=<db> \
    -e MYSQL_USER=<user> \
    -e MYSQL_PASSWORD=<pwd> \
    mysql:8.0.31
```

## 5 Nacos 环境

### 5.1 安装

#### 5.1.1 通过 Docker 安装

拉取镜像
```bash
docker pull nacos/nacos-server:v2.3.0
```
创建环境变量文件
```sh
# nacos.env

MODE=standalone
NACOS_AUTH_ENABLE=true
SPRING_DATASOURCE_PLATFORM=mysql
MYSQL_SERVICE_HOST=yourhost
MYSQL_SERVICE_DB_NAME=<db>
MYSQL_SERVICE_PORT=3306
MYSQL_SERVICE_USER=<user>
MYSQL_SERVICE_PASSWORD=<pwd>
MYSQL_SERVICE_DB_PARAM=characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
NACOS_AUTH_IDENTITY_KEY=2222
NACOS_AUTH_IDENTITY_VALUE=2xxx
NACOS_AUTH_TOKEN=SecretKey012345678901234567890123456789012345678901234567890123456789
```
创建容器并启动
```bash
docker run -d \
    --name nacos-server \
    -p 8848:8848 \
    -p 9848:9848 \
    -v ./nacos/logs:/home/nacos/logs \
    --env-file nacos.env \
    nacos/nacos-server:v2.3.0
```

### 5.2 使用

启动成功后使用 nacos/nacos 访问 [http://localhost:8848/nacos](http://localhost:8848/nacos) 即可进入 web 管理页面

## 6 防火墙配置

### 6.1 获取所有端口

```bash
firewall-cmd --list-all
```

### 6.2 开放端口

```bash
firewall-cmd --zone=public --add-port=8080/tcp --permanent
```

### 6.3 关闭端口

```bash
firewall-cmd --remove-port=8080/tcp --permanent
```

### 6.4 刷新端口

```bash
firewall-cmd --reload
```