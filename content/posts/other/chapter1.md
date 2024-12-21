---
title: "ArchLinux 安装教程（双系统）"
date: 2023-09-12T10:00:00+08:00
categories: ["other"]
draft: false
---

## 介绍

[Arch is the best](https://wiki.archlinux.org/title/Arch_is_the_best)

## 安装

### 1 前置准备

#### 1.1 获取安装镜像

从 [mirrors](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/) 下载 ArchLinux 镜像

#### 1.2 准备安装介质

推荐使用 [rufus](https://rufus.ie/) 工具烧录启动盘

#### 1.3 启动到 live 环境

#### 1.4 连接到互联网（wifi）

打开 iwctl
```bash
iwctl
```
获取设备列表
```bash
device list
```
扫描网络（默认设备 wlan0）
```bash
station wlan0 scan
```
获取可用网络列表
```bash
station wlan0 get-networks
```
连接到网络
```bash
station wlan0 connect <SSID>  # SSID wifi 名称
```
输入密码并退出 iwctl

检查网络连接
```bash
ping -c4 www.baidu.com
```

#### 1.5 更新系统时间

使用 timedatectl 确保系统时间是准确的
```bash
timedatectl
```

#### 1.6 创建磁盘分区

查看磁盘分区情况
```bash
fdisk -l
```
使用 cfdisk 操作磁盘分区
```bash
cfdisk /dev/<the_disk_to_be_partitioned>  # the_disk_to_be_partitioned 要被分区的磁盘
```
创建 root 分区（100G 或自定义大小）和 EFI 分区（300M）即可，如果安装双系统可以不建 EFI 分区，使用已有的 EFI 分区，其他分区可参考 [Example layouts](https://wiki.archlinux.org/title/Partitioning#Example_layouts) 中的分区方案

> 本文不使用交换分区，使用[交换文件](#33-创建交换文件)代替

#### 1.7 格式化分区

格式化 root 分区为 Ext4
```bash
mkfs.ext4 /dev/<root_partition>  # root_partition 根分区
```
格式化 EFI 分区为 Fat32（双系统忽略此步骤）
```bash
mkfs.fat -F 32 /dev/<efi_system_partition>  # efi_system_partition EFI 分区
```

#### 1.8 挂载分区

将 root 分区挂载到 /mnt
```bash
mount /dev/<root_partition> /mnt  # root_partition 根分区
```
将 EFI 分区挂载到 /mnt/boot
```bash
mount --mkdir /dev/<efi_system_partition> /mnt/boot  # efi_system_partition EFI 分区
```

### 2 系统安装

#### 2.1 选择镜像站

添加合适的 mirror 到 mirrorlist 的第一条
```sh
# /etc/pacman.d/mirrorlist

Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```
更新源
```bash
pacman -Syy
```

#### 2.2 安装必需的软件包

使用 pacstrap 安装 base 软件包和 Linux 内核以及常规硬件的固件
```bash
pacstrap -K /mnt base linux linux-firmware intel-ucode vim
```

### 3 配置系统

#### 3.1 生成 fstab 文件

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

#### 3.2 chroot 到新安装的系统

```bash
arch-chroot /mnt
```

#### 3.3 创建交换文件

使用 dd 创建一个 2G 的交换文件
```bash
dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress
```
为交换文件设置权限
```bash
chmod 0600 /swapfile
```
格式化交换文件
```bash
mkswap -U clear /swapfile
```
启用交换文件
```bash
swapon /swapfile
```
在 fstab 中添加交换文件条目
```sh
# /etc/fstab

/swapfile none swap defaults 0 0
```

#### 3.4 设置时区

通过如下命令设置时区
```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
然后运行 hwclock 生成 /etc/adjtime
```bash
hwclock --systohc
```

#### 3.5 区域和本地化设置

编辑 /etc/locale.gen，然后取消掉 en_US.UTF-8 UTF-8 前面的注释

生成 locale 信息
```bash
locale-gen
```
创建 locale.conf 文件，并添加 LANG 变量
```sh
# /etc/locale.conf

LANG=en_US.UTF-8
```

#### 3.6 网络配置

创建 hostname 文件
```sh
# /etc/hostname

myhostname  # 主机名
```
安装网络程序
```bash
pacman -S dhcpcd networkmanager
```

#### 3.7 使用 passwd 设置 root 密码

```bash
passwd
```

#### 3.8 安装引导程序

使用 bootctl 生成引导
```bash
bootctl install
```
添加 arch 启动项
```bash
cp /usr/share/systemd/bootctl/arch.conf /boot/loader/entries/arch.conf
```
添加 arch 对应的磁盘信息
```bash
echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/root_partition（根分区）) rw" >> /boot/loader/entries/arch.conf
```
编辑 arch.conf 删除多余的 options

编辑启动选单 loader.conf 配置
```sh
# /boot/loader/loader.conf

default  arch.conf
timeout  4
console-mode  max
editor  no
```
至此系统安装完毕，在重启计算机前可选择[安装后的工作](#安装后的工作)，或后续执行

### 4 重新启动计算机

#### 4.1 退出 chroot 环境

```bash
exit
```

#### 4.2 解除挂载

```bash
umount -R /mnt
```

#### 4.3 关机

```bash
shutdown -h now
```

## 安装后的工作

### 1 添加用户

安装 sudo（或者 base-devel ）
```bash
pacman -S sudo
```
使用 useradd 添加用户
```bash
useradd -m -G wheel <username>  # username 用户名
```
使用 passwd 设置用户密码
```bash
passwd <username>  # username 用户名
```
编辑 /etc/sudoers，然后取消掉 %wheel ALL=(ALL:ALL) ALL 前面的注释

### 2 安装 Aur 助手（yay）

安装必要依赖
```bash
pacman -S git base-devel  # base-devel 已安装则忽略
```
安装 yay
```bash
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```

### 3 安装桌面环境

#### 3.1 Gnome 桌面（推荐）

安装 gnome 桌面
```bash
pacman -S gnome
```
启用显示管理器
```bash
systemctl enable gdm
```

#### 3.2 Hyprland 桌面

安装桌面及必要组件
```bash
yay -S hyprland kitty dolphin wofi
```
使用命令 `Hyprland` 或显示管理器启动

### 4 中文环境及输入法

安装中文字体，推荐 noto-fonts-cjk
```bash
pacman -S noto-fonts-cjk
```
安装中文输入法（fcitx5 基础包，fcitx5-chinese-addons 拼音输入，fcitx5-qt fcitx5-gtk 输入法模块）
```bash
pacman -S fcitx5 fcitx5-chinese-addons fcitx5-qt fcitx5-gtk
```
编辑 /etc/environment，添加如下配置（fcitx5 配置）
```sh
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```
配置输入法，配置文件位于 ~/.config/fcitx5，可使用 GUI 配置（需安装 fcitx5-configtool）

### 5 其他

#### 5.1 浏览器

```bash
yay -S microsoft-edge-stable-bin
```

#### 5.2 开发工具
```bash
yay -S visual-studio-code-bin
```

#### 5.3 远程连接

安装
```bash
pacman -S openssh
```
生成秘钥
```bash
ssh-keygen -A
```
运行
```bash
/usr/bin/sshd
```

#### 5.4 联网

命令行
```bash
nmcli device wifi connect <SSID_or_BSSID> password <password>
```
图形化页面
```bash
nmtui
```

## 问题与解决

### 1 引导问题

linux 引导默认在 windows 引导之后，需要手动在 bios 中切换引导启动顺序

### 2 时间设置问题

双系统时间不一致问题

linux 下可执行如下命令修改
```bash
timedate set-local-rtc 1
```

> 或者修改 windows 注册表，将 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation\RealTimeIsUniversal 键值设置为 1

### 3 iwd 联网失败问题

修改 main.conf
```sh
# /etc/iwd/main.conf

[General]
EnableNetworkConfiguration=true
```
启动 systemd-resolved 服务
```bash
systemctl start systemd-resolved
```

### 4 虚拟机配置

设置 > 选项 > 高级 > 固件类型 > UEFI