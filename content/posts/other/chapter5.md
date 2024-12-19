---
title: "Termux 的安装与使用"
date: 2024-10-24T16:00:00+08:00
categories: ["other"]
draft: false
---

## 介绍

Termux 是一个适用于 Android 的终端模拟器，其环境类似于 Linux 环境。无需 Root 或设置即可使用。

## 1 安装

### 1.1 获取安装包

从 [github](https://github.com/termux/termux-app/releases) 获取安装包

### 1.2 安装

### 1.3 换源

执行换源命令
```
termux-change-repo
```
选择 Single mirror，空格 + 回车

选择 Tsinghua 源，空格 + 回车

## 2 配置

### 2.1 用户名及密码

获取当前用户
```
whoami
```
设置密码
```
passwd
```

### 2.2 开启远程访问

安装 ssh
```
pkg install openssh
```
运行 ssh
```
sshd
```
获取 ip
```
ifconfig
```

### 2.3 桌面环境

安装桌面环境及 vnc 服务
```
pkg install x11-repo
pkg install tigervnc xfce4
```
启动 vnc 服务
```
vncserver :1
```
设置 vnc 密码

从 Vnc Viewer 创建连接，输入 localhost:1

此时窗口是空的

返回 termux 终端，执行命令
```
export DISPLAY=:1
xfce4-session &
```

回到 Vnc Viewer 正常显示桌面

关闭 vnc 服务
```
vncserver -kill :1
```

## 3 其他

### 3.1 Java 环境

安装
```
pkg install openjdk-17
```
验证
```
java -versoin
```

### 3.2 Redis 环境

安装
```
pkg install redis
```
启动
```
redis-server /data/data/com.termux/files/usr/etc/redis.conf
```
如报错：WARINING Your kernel has a bug xxx，则修改 redis.conf
```
...
# 放开注释
ignore-warnings ARM64-COW-BUG
```

### 3.3 MariaDB 环境

安装
```
pkg install mariadb
```
验证
```
mysql --version
```
启动
```
mysqld &
```

> 传统 Linux 目录 /etc 对应 Termux 目录 /data/data/com.termux/files/usr/etc