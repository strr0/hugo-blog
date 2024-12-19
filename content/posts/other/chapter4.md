---
title: "ArchLinux 下 Hyprland 配置"
date: 2024-11-02T10:00:00+08:00
categories: ["other"]
draft: false
---

## 介绍

Hyprland 是一款基于 Wayland 的平铺式窗口管理器

## 1 基础

### 1.1 安装 Hyprland

安装依赖
```
yay -S hyprland
```
复制配置文件
```
mkdir -pv ~/.config/hypr
cp /usr/share/hyprland/hyprland.conf ~/.config/hypr
```

### 1.2 配置

快捷键绑定
```
bind = SUPER, T, exec, foot --override shell=fish
```
设置环境变量
```
env = XCURSOR_SIZE, 24
```
启动
```
exec-once = wl-paste --watch cliphist store
```

## 2 其他

### 2.1 终端

① kitty
```
yay -S kitty
```

② fish
```
yay -S fish
```

### 2.2 应用启动器

① wofi
```
yay -S wofi
```

② rofi
```
yay -S rofi
```

③ fuzzel
```
yay -S fuzzel
```

### 2.3 壁纸

① swaybg

安装
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

② hyprpaper

安装
```
yay -S hpyrpaper
```
设置壁纸
```
~/.config/hypr/hyprpaper.conf

preload = /home/me/amongus.png
wallpaper = monitor, /home/me/amongus.png
```

### 2.4 状态栏

安装
```
yay -S waybar
```
设置状态栏
```
~/.config/hypr/hyprland.conf

exec-once = waybar
```

### 2.5 音频

安装
```
yay -S wireplumber
```
启动
```
systemctl --user start wireplumber
```
获取设备状态
```
wpctl status
```
调节音频音量
```
wpctl set-volume @DEFAULT_AUDIO_SINK@ 5%+
```
设置麦克风静音
```
wpctl set-mute @DEFAULT_AUDIO_SOURCE@ toggle
```
若未识别音频设备，尝试修改开启 alsa.use-ucm
```
~/.config/wireplumber/wireplumber.conf.d/alsa-vm.conf

monitor.alsa.properties = {
  # Use ALSA-Card-Profile devices. They use UCM or the profile
  # configuration to configure the device and mixer settings.
  # alsa.use-acp = true
  # Use UCM instead of profile when available. Can be disabled
  # to skip trying to use the UCM profile.
  alsa.use-ucm = true
}
```

### 2.6 亮度

安装
```
yay -S light
```
增加亮度
```
light -A 5
```
减少亮度
```
light -U 5
```
设置亮度
```
light -S 100
```
若无法调节亮度，尝试将用户加入组
```
usermod -aG video <USERNAME>
```

### 2.7 输入法

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

### 2.8 文件管理

① dolphin
```
yay -S dolphin
```

② nautilus
```
yay -S nautilus
```

### 2.9 复制及截图

安装复制工具
```
yay -S cliphist wl-clipboard
```
开启剪贴板监听
```
exec-once = wl-paste --watch cliphist store
```
安装截图工具
```
yay -S slurp grim
```
配置截图快捷键
```
~/.config/hypr/hyprland.conf

$screen_file=${HOME}/Pictures/ScreenShot/screen_shot_$(date +%Y%m%d%H%M%S).png
bind = , Print, exec, grim $screen_file
bind = SHIFT, Print, exec, grim -g "$(slurp)" $screen_file
bind = ALT, A, exec, grim -g "$(slurp)" - | wl-copy
```

### 2.10 锁屏

安装
```
yay -S hyprlock
```
配置
```
~/.config/hypr/hyprlock.conf

background {
    monitor =
    path = /home/me/someImage.png   # supports png, jpg, webp (no animations, though)
    color = rgba(25, 20, 20, 1.0)

    # all these options are taken from hyprland, see https://wiki.hyprland.org/Configuring/Variables/#blur for explanations
    blur_passes = 0 # 0 disables blurring
    blur_size = 7
    noise = 0.0117
    contrast = 0.8916
    brightness = 0.8172
    vibrancy = 0.1696
    vibrancy_darkness = 0.0
}

label {
    monitor =
    text = Hi there, $USER
    text_align = center # center/right or any value for default left. multi-line text alignment inside label container
    color = rgba(200, 200, 200, 1.0)
    font_size = 25
    font_family = Noto Sans
    rotate = 0 # degrees, counter-clockwise

    position = 0, 80
    halign = center
    valign = center
}

input-field {
    monitor =
    size = 200, 50
    outline_thickness = 3
    dots_size = 0.33 # Scale of input-field height, 0.2 - 0.8
    dots_spacing = 0.15 # Scale of dots' absolute size, -1.0 - 1.0
    dots_center = false
    dots_rounding = -1 # -1 default circle, -2 follow input-field rounding
    dots_fade_time = 200 # Milliseconds until a dot fully fades in
    dots_text_format = # Text character used for the input indicator. Leave empty for a rectangle that will be rounded via dots_rounding (default).
    outer_color = rgb(151515)
    inner_color = rgb(200, 200, 200)
    font_color = rgb(10, 10, 10)
    font_family = Noto Sans # Font used for placeholder_text, fail_text and dots_text_format.
    fade_on_empty = true
    fade_timeout = 1000 # Milliseconds before fade_on_empty is triggered.
    placeholder_text = <i>Input Password...</i> # Text rendered in the input box when it's empty.
    hide_input = false
    rounding = -1 # -1 means complete rounding (circle/oval)
    check_color = rgb(204, 136, 34)
    fail_color = rgb(204, 34, 34) # if authentication failed, changes outer_color and fail message color
    fail_text = <i>$FAIL <b>($ATTEMPTS)</b></i> # can be set to empty
    fail_timeout = 2000 # milliseconds before fail_text and fail_color disappears
    fail_transition = 300 # transition time in ms between normal outer_color and fail_color
    capslock_color = -1
    numlock_color = -1
    bothlock_color = -1 # when both locks are active. -1 means don't change outer color (same for above)
    invert_numlock = false # change color if numlock is off
    swap_font_color = false # see below

    position = 0, -20
    halign = center
    valign = center
}
```