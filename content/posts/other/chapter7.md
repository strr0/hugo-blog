---
title: "PostmarketOS 安装"
date: 2024-11-09T16:00:00+08:00
categories: ["other"]
draft: false
---

## 介绍

Linux 手机发行版

## 1 准备

- 工作站至少有 10G 的空闲空间
- 已阅读设备的 wiki 页
- 保证设备电量充足
- 设备已解锁 bootloader
- 已备份设备数据

## 2 开始

### 2.1 安装构建工具

更新 python 版本
```
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python3.12
```
修改 link
```
cd /usr/bin
mv python3 python3.bak
ln -s python3.12 python3
```
设置环境变量
```
echo export PATH=\$PATH:\$HOME/.local/bin >> ~/.bashrc
source ~/.bashrc
```
安装
```
git clone https://gitlab.postmarketos.org/postmarketOS/pmbootstrap.git
cd pmbootstrap
mkdir -p ~/.local/bin
ln -s "$PWD/pmbootstrap.py" ~/.local/bin/pmbootstrap
```
验证
```
pmbootstrap --version
```

> 提示 ModuleNotFoundError: No module named 'tomllib'，则安装 python3-tomli

### 2.2 初始化

环境配置包括：

- 工作路径
- 滚动版或稳定版
- 设备品牌和型号
- 桌面环境选择
- 用户信息
- 时区

初始化环境配置
```
pmbootstrap init
```
配置镜像
```
pmbootstrap config mirrors.alpine https://mirrors.tuna.tsinghua.edu.cn/alpine/
pmbootstrap config mirrors.pmaports https://mirrors.tuna.tsinghua.edu.cn/postmarketOS/
```

### 2.3 生成镜像

#### 2.3.1 生成常规镜像

运行命令
```
pmbootstrap install
```

#### 2.3.2 生成并刷入

使用 lsblk 找到设备路径，SD 卡通常在 /dev/mmcblk，可以通过如下命令获取
```
sudo fdisk -l /dev/[yourdevice]
```
运行命令（/dev/mmcblk... 替换为实际获取的路径）
```
pmbootstrap install --disk=/dev/mmcblk...
```

#### 2.3.3 开启磁盘加密

运行命令
```
pmbootstrap install --fde
```

### 2.4 刷入镜像

将设备启动到 fastboot 模式

刷入 rootfs
```
pmbootstrap flasher flash_rootfs
```
刷入 kernel
```
pmbootstrap flasher flash_kernel
```