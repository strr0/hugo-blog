---
title: "Halium 编译"
date: 2024-11-04T16:00:00+08:00
categories: ["other"]
draft: false
---

## 介绍

Halium 是一个合作项目，是一个致力于统一硬件抽象层项目，并通过预安装到安卓设备上，达到运行 GNU/Linux 的目的。

## 1 准备

### 1.1 安卓设备

#### 1.1.1 源码

设备需要有 Linux 内核源码支持（需要基于此版本的 LineageOS 支持）

#### 1.1.2 内核

当前版本的 Halium 需要高于 3.10.0 的 Linux 内核（一些 Halium 发行版可能使用低于 3.4 的内核，如 Ubuntu Touch）

#### 1.1.3 内存

至少 1G 内存（2G 及以上可以有更好的体验）

#### 1.1.4 磁盘

至少 16G 磁盘空间（低于此可能没有足够的空间）

### 1.2 编译环境

#### 1.2.1 Debian 系列

##### 1.2.1.1 更新软件包

如果使用的是 amd64 架构，需要开启使用 i386 架构
```
sudo dpkg --add-architecture i386
```
更新软件包
```
sudo apt update
```
***

Debian (Stretch or newer) / Ubuntu (16.04 or 18.04)

下载所需依赖
```
sudo apt install git gnupg flex bison gperf build-essential \
  zip bzr curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
  libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
  libgl1-mesa-dev g++-multilib mingw-w64-i686-dev tofrodos \
  python-markdown libxml2-utils xsltproc zlib1g-dev:i386 schedtool \
  repo liblz4-tool bc lzop imagemagick libncurses5 rsync
```
***

Ubuntu (20.04 or newer)

下载所需依赖
```
sudo apt install git gnupg flex bison gperf build-essential \
  zip bzr curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
  libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
  libgl1-mesa-dev g++-multilib mingw-w64-i686-dev tofrodos \
  python-markdown libxml2-utils xsltproc zlib1g-dev:i386 schedtool \
  liblz4-tool bc lzop imagemagick libncurses5 rsync \
  python-is-python3
```

##### 1.2.1.4 git repo

在 home 目录下创建 bin 目录，并添加到 PATH 环境变量
```
mkdir -p ~/bin
echo export PATH=\$PATH:\$HOME/bin >> ~/.bashrc
source ~/.bashrc
```
***

下载 git repo 脚本（google）
```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+rx ~/bin/repo
```
***

下载 git repo 脚本（中科大源）
```
curl -sSL  'https://gerrit-googlesource.proxy.ustclug.org/git-repo/+/master/repo?format=TEXT' |base64 -d > ~/bin/repo
chmod a+x ~/bin/repo
```
更新 REPO_URL
```
echo export REPO_URL='https://gerrit-googlesource.proxy.ustclug.org/git-repo' >> ~/.bashrc
source ~/.bashrc
```

#### 1.2.2 Arch 系列

如果使用的是 amd64 架构，需要在 /etc/pacman.conf 中添加 multilib 源，以便使用 i686 的包。

获取依赖代码
```
git clone https://aur.archlinux.org/halium-devel.git
```
编译及安装
```
cd halium-devel && makepkg -i
```

## 2 开始

### 2.1 获取源码

设置 git 用户名及邮箱
```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```
创建源码路径
```
mkdir halium && cd halium
```
获取源码（以 halium-7.1 为例）
```
repo init -u https://github.com/Halium/android -b halium-7.1 --depth=1
```
***

同步源码树（google）
```
repo sync -c -j 16
```
***

替换源码 remote
```
git config --global url.https://mirrors.ustc.edu.cn/aosp/.insteadof https://android.googlesource.com
```
同步源码树（中科大源，sync 默认使用 4 个并发连接，请勿使用 -j 参数增加并发连接数）
```
repo sync -c
```

### 2.2 配置 Manifest

#### 2.2.1 获取 dependencies

在 [LineageOS](https://github.com/LineageOS) 找到设备对应的仓库的 cm.dependencies 或 lineage.dependencies

#### 2.2.2 创建文件

创建文件 halium/devices/manifests/[manufacturer]_[device].xml
```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
</manifest>
```

#### 2.2.3 添加设备信息

在 manifest 标签中添加如下配置
```
<project path="device/[manufacturer] / [device]" name="[repository name]" remote="[remote]" revision="[revision]" />
```

#### 2.2.4 添加依赖信息

根据 cm.dependencies 或 lineage.dependencies 填充配置
```
<project path="[target_path]" name="[repository]" remote="[remote]" revision="[revision]" />
```

#### 2.2.5 添加 Vendor

在 [TheMuppets](https://github.com/TheMuppets) 找到设备对应的配置添加到 vendor/ 目录中

#### 2.2.6 添加 remote

如需添加 remote，可以在 .repo/manifest.xml 下添加
```
<remote name="mun"
    fetch="https://github.com/MyUserName"
    revision="cm-14.1" />
```
***

halium-7.1 默认 remote
|  Remote Name  |               Remote Description, URL                                      |
| :-----------: | :------------------------------------------------------------------------: |
|  aosp         |  Android Open Source Project, https://android.googlesource.com             |
|  los          |  LineageOS, http://github.com/LineageOS                                    |
|  hal          |  Halium (link to GitHub root for legacy reasons), http://github.com        |
|  them         |  TheMuppets, http://github.com/TheMuppets                                  |
|  them2        |  TheMuppets (for some xiaomi vendor repos) https://gitlab.com/the-muppets  |
***

halium-5.1 默认 remote
|  Remote Name  |               Remote Description, URL                                          |
| :-----------: | :----------------------------------------------------------------------------: |
|  phablet      |  Canonical Ubuntu Phone compatibility, https://code-review.phablet.ubuntu.com  |
|  aosp         |  Android Open Source Project, https://android.googlesource.com                 |
|  cm           |  CyanogenMod, https://github.com/CyanogenMod                                   |
|  ubp          |  UBports (link to GitHub root for legacy reasons), https://github.com          |
|  halium       |  Halium (link to GitHub root for legacy reasons), https://github.com           |
|  ab2ut        |  Vendor blobs for UBports builds, https://github.com/ab2ut                     |

#### 2.2.7 获取相关源码

manifest 填充完毕后，执行命令（DEVICE 替换成设备型号）
```
./halium/devices/setup DEVICE
```

### 2.3 构建源码

#### 2.3.1 初始化

初始化环境变量
```
source build/envsetup.sh
```
成功则返回如下信息
```
including device/lge/bullhead/vendorsetup.sh
including vendor/cm/vendorsetup.sh
including sdk/bash_completion/adb.bash
including vendor/cm/bash_completion/git.bash
including vendor/cm/bash_completion/repo.bash
```

#### 2.3.2 构建环境变量

Halium-5.1

执行命令
```
lunch
```
得到如下信息
```
1. aosp_arm64-eng   4. aosp_mips-eng     7. cm_bacon-eng
2. aosp_arm-eng     5. aosp_x86_64-eng   8. cm_bacon-user
3. aosp_mips64-eng  6. aosp_x86-eng      9. cm_bacon-userdebug
```
选择设备 cm_[your device]-userdebug
***

Halium-7.1

执行命令
```
breakfast [codename]
```

#### 2.3.3 修改内核配置

内核配置路径（路径一般在 arch/arm/configs/<CONFIG> 或 arch/arm64/configs/<CONFIG> 下）
```
grep "TARGET_KERNEL_CONFIG" device/<VENDOR>/<CODENAME>/BoardConfig.mk
```
检查配置及修复
```
./halium/halium-boot/check-kernel-config path/to/my/defconfig -w
```

~~提示 WARNING 可以忽略，修改 ERROR 的即可~~

~~最后 CONFIG_IKCONFIG 和 CONFIG_IKCONFIG_PROC 需要设置为 y，否则设备将无法启动~~

#### 2.3.4 Ubuntu Touch 配置

在 BoardConfig.mk 配置中添加（~/halium/device/<vendor>/<model_codename>/BoardConfig.mk）
```
BOARD_KERNEL_CMDLINE += console=tty0
```
如果在 boot 之后不能 ssh 连接，尝试修改内核配置
```
CONFIG_CMDLINE="console=tty0"
CONFIG_CMDLINE_EXTEND=y
```

#### 2.3.5 挂载点修复

检查设备型号是否在挂载点修复脚本中（<BUILDDIR>/halium/hybris-boot/fixup-mountpoints），如果不在则需要手动添加：

1 找到设备的 fstab（如 Moto G5 Plus，在 device/motorola/potter/rootdir/etc 的 fstab.qcom）

2 使用 adb shell 进入并获取 root 权限

3 创建配置
```
"[codename]")
    sed -i \
        [replacements, one per line]
        "$@"
    ;;
```

4 对于所有非 auto emmc swap 的条目，执行 readlink -f [src] 获取返回值

5 用 4 中的到的结果替换 3 中的每一行
```
  -e 's [src] [return] ' \
```

#### 2.3.6 添加 Hybris 补丁（halium 9 以上）

```
hybris-patches/apply-patches.sh --mb
```

#### 2.3.7 构建镜像

生成构建工具
```
mka mkbootimg
```
构建镜像
```
export USE_HOST_LEX=yes
mka halium-boot
mka systemimage
```

> 如果遇到 cc1: error: -Werror=stringop-truncation: no option -Wstringop-truncation，可能是由于编译器版本太低，尝试更新 gcc

#### 2.3.7 问题

make: *** No rule to make target '/home/ubuntu/halium/out/target/product/n7100/obj/SHARED_LIBRARIES/libandroid_runtime_intermediates/export_includes', needed by '/home/ubuntu/halium/out/target/product/n7100/obj/STATIC_LIBRARIES/libsecosal_intermediates/import_includes'.  Stop.