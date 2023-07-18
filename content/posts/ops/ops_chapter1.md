---
title: "适用于Windows的Linux子系统"
date: 2023-04-06T19:30:00+08:00
draft: false
---

### 背景
作为一个开发人员，做好本职开发之外，掌握一些框架组件，诸如消息队列、缓存、容器技术等等，也是同样重要的。你需要去安装它们，配置它们的环境，当然你可以选择直接在物理机上安装，但是有时候这么做并不是一个最好的选择。因为在物理机上安装有可能会污染你当前的开发环境配置，再有大部分的框架组件是基于linux环境的，虽然也有对windows做适配，但是它们是更亲近于linux的，在windows上运行可能会遇到意想不到的问题，而在linux上有更健全的文档，遇到的问题也有更专业更详细的解答。因此搭建一个linux虚拟机环境是一个非常不错的选择。
### 优缺点
常见的虚拟机如VMware、VirtualBox等等，但是我此次选择的是wsl。wsl是一个在Windows上能够运行原生Linux二进制可执行文件的兼容层，目前也在迅速发展中，能否成为趁手的虚拟机环境还是看个人喜好，但我认为是值得体验一下的。相比于传统虚拟机，它启动速度更快，而且与windows的交互更加丝滑，挂载windows文件夹操作简单，使用上感觉就是在windows上一样，然后它与宿主机是共享ip端口的，所以安装到虚拟机上的软件，也可以在windows上通过localhost访问，体验特别棒。缺点就是，使用它需要开启Hyper-V，而Hyper-V会与其他虚拟机环境冲突，所以在开启Hyper-V后，会导致VMware、安卓虚拟机等不能使用，若要使用则需要关闭Hyper-V。
### 安装
Windows 10 版本 2004 及更高版本（内部版本 19041 及更高版本）或 Windows 11 使用自动安装，否则使用手动安装  
1. 自动安装：  
执行安装命令  
```bash
wsl --install
```
2. 手动安装：   
①启用“适用于 Linux 的 Windows 子系统”可选功能
②检查运行 WSL 2 的要求  
若要更新到 WSL 2，需要运行 Windows 10  
对于 x64 系统：版本 1903 或更高版本，内部版本为 18362 或更高版本。  
对于 ARM64 系统：版本 2004 或更高版本，内部版本为 19041 或更高版本。  
或 Windows 11。
③启用“虚拟机平台”可选功能  
④下载 Linux 内核更新包  
下载最新包：[适用于 x64 计算机的 WSL2 Linux 内核更新包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)  
运行上一步中下载的更新包。  
⑤将 WSL 2 设置为默认版本  
```bash
wsl --set-default-version 2
```
⑥安装 Linux 分发  
从商店下载或点击[Ubuntu](https://aka.ms/wslubuntu)、[Debian GNU/Linux](https://aka.ms/wsl-debian-gnulinux)、[Kali Linux](https://aka.ms/wsl-kali-linux-new)、[Fedora Remix for WSL](https://github.com/WhitewaterFoundry/WSLFedoraRemix/releases/)
### wsl基础操作
查看wsl版本
```
wsl -l -v
```
设置wsl版本
```
wsl --set-version {所选的linux分支} 2
```
启动wsl
```
wsl -d {所选的linux分支}
```
关闭wsl
```
wsl --shutdown
```
导入wsl
```
wsl --import {镜像名称} {安装路径} {镜像路径} --version 2 
```
wsl子系统安装列表
```
wslconfig /l
```
卸载子系统
```
wslconfig /u {所选的linux分支}
```