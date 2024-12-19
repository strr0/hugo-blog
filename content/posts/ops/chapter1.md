---
title: "适用于 Windows 的 Linux 子系统"
date: 2023-04-06T19:30:00+08:00
categories: ["operations"]
draft: false
---

## 介绍

wsl 是一个在 Windows 上能够运行原生 Linux 二进制可执行文件的兼容层，相比于传统虚拟机，它启动速度更快，而且与 windows 的交互更加丝滑，挂载 windows 文件夹操作简单。缺点就是，使用它需要开启 Hyper-V，而 Hyper-V 会与其他虚拟机环境冲突，所以在开启 Hyper-V 后，会导致 VMware、安卓虚拟机等不能使用，若要使用则需要关闭 Hyper-V。

## 1 安装

Windows 10 版本 2004 及更高版本（内部版本 19041 及更高版本）或 Windows 11 使用自动安装，否则使用手动安装

### 1.1 自动安装

执行安装命令
```bash
wsl --install
```

### 1.2 手动安装

#### 1.2.1 启用“适用于 Linux 的 Windows 子系统”可选功能

#### 1.2.2 检查运行 WSL 2 的要求

若要更新到 WSL 2，需要运行 Windows 10  
对于 x64 系统：版本 1903 或更高版本，内部版本为 18362 或更高版本。  
对于 ARM64 系统：版本 2004 或更高版本，内部版本为 19041 或更高版本。  
或 Windows 11。

#### 1.2.3 启用“虚拟机平台”可选功能

#### 1.2.4 下载 Linux 内核更新包

下载最新包：[适用于 x64 计算机的 WSL2 Linux 内核更新包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)  
运行上一步中下载的更新包。

#### 1.2.5 将 WSL 2 设置为默认版本

```bash
wsl --set-default-version 2
```

#### 1.2.6 安装 Linux 分发

从商店下载或点击 [Ubuntu](https://aka.ms/wslubuntu)、[Debian GNU/Linux](https://aka.ms/wsl-debian-gnulinux)、[Kali Linux](https://aka.ms/wsl-kali-linux-new)、[Fedora Remix for WSL](https://github.com/WhitewaterFoundry/WSLFedoraRemix/releases/)

## 2 基础操作

查看 wsl 版本
```
wsl -l -v
```
设置 wsl 版本
```
wsl --set-version {所选的linux分支} 2
```
启动 wsl
```
wsl -d {所选的linux分支}
```
关闭 wsl
```
wsl --shutdown
```
导入 wsl
```
wsl --import {镜像名称} {安装路径} {镜像路径} --version 2 
```
wsl 子系统安装列表
```
wslconfig /l
```
卸载子系统
```
wslconfig /u {所选的linux分支}
```