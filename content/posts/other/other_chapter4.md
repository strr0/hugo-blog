---
title: "ArchLinux下Hyprland配置"
date: 2023-10-19T10:00:00+08:00
draft: false
---

## 介绍
Hyprland 是一款基于 Wayland 的平铺式窗口管理器

## 安装及配置
### 1 安装 Hyprland 及基础组件  
安装依赖
```
yay -S hyprland kitty dolphin wofi
```
复制配置文件
```
mkdir -pv ~/.config/hypr
cp /usr/share/hyprland/hyprland.conf ~/.config/hypr
```

### 2 壁纸
安装 swaybg
```
yay -S swaybg
```
设置壁纸
```
~/.config/hypr/hyprland.conf

# 单壁纸
$wallpaper_path=<你放壁纸的完整路径>
exec-once=swaybg -i $wallpaper_path -m fill

# 多壁纸
$wallpaper_dir=<你存放壁纸的目录>
exec-once=swaybg -i $(find $wallpaper_dir -type f | shuf -n 1) -m fill
```

### 3 状态栏
安装 waybar
```
yay -S waybar
```
设置状态栏
```
~/.config/hypr/hyprland.conf

exec-once=waybar
```

### 4 输入法
安装 fcitx5
```
yay -S fcitx5 fcitx5-chinese-addons fcitx5-qt fcitx5-gtk
```
修改配置
```
/etc/environment

GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```
添加输入法（fcitx5-configtool）  
设置输入法
```
~/.config/hypr/hyprland.conf

exec-once=fcitx5 --replace -d
```

### 5 截图
安装截图工具
```
yay -S cliphist wl-clipboard pipewire wireplumber slurp grim
```
配置截图快捷键
```
~/.config/hypr/hyprland.conf

$screen_file=${HOME}/Pictures/ScreenShot/screen_shot_$(date +%Y%m%d%H%M%S).png
bind = , Print, exec, grim $screen_file
bind = SHIFT, Print, exec, grim -g "$(slurp)" $screen_file
bind = ALT, A, exec, grim -g "$(slurp)" - | wl-copy
```