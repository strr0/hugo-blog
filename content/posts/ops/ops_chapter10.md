---
title: "基础环境配置"
date: 2024-07-12T10:00:00+08:00
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
```
sudo mkdir /usr/local/java
sudo tar -xzvf jdk-17_linux-x64_bin.tar.gz -C /usr/local/java
```

### 1.3 环境变量配置  
修改环境变量  
```
/etc/profile

JAVA_HOME=/usr/local/java/jdk-17.0.8
export PATH=$PATH:$JAVA_HOME/bin
```
应用环境变量  
```
source /etc/profile
```

### 1.4 验证  
运行命令验证  
```
java -version
```

### 1.5 问题与修复
Cannot load from short array because "sun.awt.FontConfiguration.head" is null  
Linux 缺失字体导致，从 jdk 1.8 的 jre/lib 复制 fontconfig.bfc 到安装的 jdk 17 的 lib 中  

## 2 Maven 环境  

### 2.1 下载  
下载[maven](https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz)  

### 2.2 安装  
解压到指定路径  
```
sudo mkdir /opt/maven
sudo tar -xzvf apache-maven-3.6.3-bin.tar.gz -C /opt/maven
```

### 2.3 环境变量配置  
修改环境变量  
```
/etc/profile

M2_HOME=/opt/maven/apache-maven-3.6.3
export PATH=$PATH:$M2_HOME/bin
```
应用环境变量  
```
source /etc/profile
```

### 2.4 验证  
运行命令验证  
```
mvn -version
```

## 3 Git 环境  

### 3.1 安装  
```
yum install git
```

### 3.2 验证
```
git --version
```