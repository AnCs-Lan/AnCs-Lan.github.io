---
title: "Hyprlock配置"
description: "一份简单的hyprlock配置文档，记录一下个人配置"
date: 2025-12-27 10:13:31
categories:
   - Arch Linux
tags:
   - Arch Linux
   - hyprland
---

# Arch Linux Hyprland + Hyprlock (Catppuccin Mocha) 配置指南

这份指南将帮助你在 Arch Linux 上配置基于 Catppuccin Mocha 主题的 Hyprlock 锁屏，并配合 Hypridle 实现自动锁屏功能。

## 1. 下载的包与依赖包

你需要安装核心锁屏程序、空闲守护进程以及推荐的字体包（用于显示美观的图标和文字）。

请在终端执行以下命令：

```bash
# 安装 Hyprlock (锁屏界面) 和 Hypridle (空闲检测)
sudo pacman -S hyprlock hypridle

# 安装字体 (推荐 JetBrainsMono Nerd Font，否则配置中的字体设置需要修改)
sudo pacman -S ttf-jetbrains-mono-nerd
```

---

## 2. 添加的配置信息

你需要创建或编辑两个配置文件。

### 文件 A: `~/.config/hypr/hyprlock.conf`
**功能**：定义锁屏界面的视觉效果（Catppuccin Mocha 主题）。

```ini
# =============================================================================
# CATPPUCCIN MOCHA PALETTE (配色变量)
# =============================================================================
$base           = rgb(30, 30, 46)      # 背景底色
$text           = rgb(205, 214, 244)   # 主文本色
$subtext        = rgb(166, 173, 200)   # 副文本色
$surface0       = rgb(49, 50, 68)      # 输入框背景
$lavender       = rgb(180, 190, 254)   # 重点色/边框
$red            = rgb(243, 139, 168)   # 错误色
$yellow         = rgb(249, 226, 175)   # 验证中颜色

# =============================================================================
# 通用设置
# =============================================================================
general {
    no_fade_in = false
    grace = 0
    disable_loading_bar = true
}

# =============================================================================
# 背景 (Background)
# =============================================================================
background {
    monitor =
    # 请修改下面的路径为你自己的壁纸路径，建议使用 Catppuccin 风格壁纸
    path = /home/username/Pictures/Wallpapers/mocha.png
    color = $base

    # 模糊效果
    blur_passes = 2
    blur_size = 7
    noise = 0.0117
    contrast = 0.8916
    brightness = 0.8172
    vibrancy = 0.1696
    vibrancy_darkness = 0.0
}

# =============================================================================
# 输入框 (Input Field)
# =============================================================================
input-field {
    monitor =
    size = 250, 50
    outline_thickness = 3
    dots_size = 0.33
    dots_spacing = 0.15
    dots_center = true
    dots_rounding = -1

    outer_color = $lavender
    inner_color = $surface0
    font_color = $text
    check_color = $yellow
    fail_color = $red

    fade_on_empty = true
    fade_timeout = 1000
    placeholder_text = <span foreground="##a6adc8">Input Password...</span>
    hide_input = false
    rounding = -1

    fail_text = <i>$FAIL <b>($ATTEMPTS)</b></i>
    fail_transition = 300

    position = 0, -20
    halign = center
    valign = center
}

# =============================================================================
# 时间 (Time)
# =============================================================================
label {
    monitor =
    text = cmd[update:1000] echo "$(date +"%H:%M")"
    color = $text
    font_size = 64
    font_family = JetBrains Mono Nerd Font Mono
    position = 0, 80
    halign = center
    valign = center
}

# =============================================================================
# 日期 (Date)
# =============================================================================
label {
    monitor =
    text = cmd[update:1000] echo "$(date +"%A, %B %d")"
    color = $subtext
    font_size = 22
    font_family = JetBrains Mono Nerd Font Mono
    position = 0, 30
    halign = center
    valign = center
}

# =============================================================================
# 问候语 (Greeting)
# =============================================================================
label {
    monitor =
    text = Hi there, $USER
    color = $text
    font_size = 14
    font_family = JetBrains Mono Nerd Font Mono
    position = 0, -80
    halign = center
    valign = center
}

# =============================================================================
# 用户头像 (Image)(可选)
# =============================================================================
image {
    monitor =
    path = /path/to/your/face.png
    size = 150 # size in pixels
    rounding = -1 # -1 = circle
    border_size = 4
    border_color = rgb(221, 221, 221)
    position = 0, 200
    halign = center
    valign = center
}
```

### 文件 B: `~/.config/hypr/hypridle.conf`
**功能**：定义何时触发锁屏（逻辑控制）。

```ini
general {
    lock_cmd = pidof hyprlock || hyprlock       # 锁屏命令
    before_sleep_cmd = loginctl lock-session    # 休眠前锁屏
    after_sleep_cmd = hyprctl dispatch dpms on  # 唤醒后打开屏幕
}

listener {
    timeout = 300                                 # 5分钟后
    on-timeout = loginctl lock-session            # 触发锁屏
}

listener {
    timeout = 330                                 # 5.5分钟后
    on-timeout = hyprctl dispatch dpms off        # 关闭显示器
    on-resume = hyprctl dispatch dpms on          # 恢复时打开显示器
}
```

---

## 3. 运行的指令

你需要将 `hypridle` 加入到 Hyprland 的自启动项中，以便它在后台运行并监控空闲时间。

**步骤：** 编辑你的 Hyprland 主配置文件（通常是 `~/.config/hypr/hyprland.conf`），在文件末尾或 `exec-once` 区域添加以下内容：

```ini
# 启动空闲守护进程 (对应 hypridle.conf)
exec-once = hypridle

# [可选] 设置快捷键手动锁屏 (Super + L)
bind = SUPER, L, exec, hyprlock
```

---

## 4. 检查成功的步骤

请按以下顺序进行验证：

1.  **重启 Hyprland**：
    退出当前会话并重新登录，或者在终端执行 `hyprctl reload`（但 `exec-once` 最好通过重启生效）。

2.  **测试界面配置 (Hyprlock)**：
    *   打开终端，直接输入命令：`hyprlock`。
    *   **现象**：屏幕应立即锁定，显示 Catppuccin Mocha 配色（淡紫色边框、深色背景），且时间显示正常。
    *   **解锁**：输入密码并回车，应该能正常返回桌面。

3.  **测试自动锁屏 (Hypridle)**：
    *   如果你设置了 `timeout = 300`，你可以暂时将其改为 `timeout = 10` (10秒) 用于测试。
    *   修改 `hypridle.conf` 后，停止鼠标和键盘操作。
    *   **现象**：等待设定时间后，屏幕应自动进入锁屏界面。

4.  **测试休眠锁屏**：
    *   在终端运行：`systemctl suspend`。
    *   **现象**：电脑挂起。当你唤醒电脑时，应该首先看到锁屏界面，而不是直接进入桌面。
