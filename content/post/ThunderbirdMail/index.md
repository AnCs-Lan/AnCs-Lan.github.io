---
title: "ThunderbirdMail"
description: "用Thunderbird配置了一个邮件管理页面"
date: 2026-01-03 00:24:23 
tags:
   - "Arch Linux"
   - "hyprland"
categories:
   - "Arch Linux"
---

# Arch Linux + Hyprland 邮件管理配置文档 (Thunderbird 方案)

本文档旨在利用 Hyprland 的“特殊工作区 (Scratchpad)”机制，实现类似“后台运行/快速呼出”的高效邮件管理体验。

## 1. 安装的包

主要安装 Thunderbird 本体及推荐的中文字体支持。

```bash
# 安装 Thunderbird
sudo pacman -S thunderbird

# (可选) 安装中文字体，防止邮件中文乱码
sudo pacman -S noto-fonts-cjk
```

---

## 2. 所作的配置

### A. 系统/窗口管理器配置
**文件路径：** `~/.config/hypr/hyprland.conf`

在配置文件中添加以下内容，以实现：
1.  强制 Thunderbird 使用 Wayland 模式（解决模糊问题）。
2.  开机自启并自动进入后台（特殊工作区）。
3.  设置快捷键呼出/隐藏，以及将窗口移回主屏幕。

```bash
# -----------------------------------------------------
# 环境变量设置
# -----------------------------------------------------
# 强制 Mozilla 软件使用 Wayland 后端，解决高分屏模糊
env = MOZ_ENABLE_WAYLAND,1

# -----------------------------------------------------
# 启动与窗口规则
# -----------------------------------------------------
# 开机自启 Thunderbird
exec-once = thunderbird

# 规则：启动时自动静默放入名为 'mail' 的特殊工作区 (Scratchpad)
windowrulev2 = workspace special:mail silent, class:^(thunderbird)$

# -----------------------------------------------------
# 快捷键绑定 (Keybindings)
# -----------------------------------------------------
# 定义主修饰键 (假设为 Super/Windows 键)
$mainMod = SUPER

# 核心功能：按 Super + M 呼出或隐藏邮件窗口 (即切换 special:mail 工作区)
bind = $mainMod, M, togglespecialworkspace, mail

# 扩展功能：按 Super + Shift + M 将当前邮件窗口“拉”回普通平铺工作区
bind = $mainMod SHIFT, M, movetoworkspace, +0
```

### B. Thunderbird 软件内部配置
**操作方式：** 启动软件后在图形界面进行设置。

1.  **布局优化 (现代化视图)**：
    *   **路径**：右上角菜单 (≡) -> `视图` -> `布局` -> 勾选 `垂直视图`。
    *   **路径**：左侧文件夹列表头部 `...` -> 勾选 `统一文件夹` (合并所有邮箱的收件箱)。

2.  **账号登录 (国内邮箱注意事项)**：
    *   **对象**：QQ邮箱 / 163网易邮箱等。
    *   **配置**：不要使用网页登录密码，**必须使用网页版设置中生成的“授权码”**。
    *   **协议**：务必选择 **IMAP** 协议。

3.  **关闭行为 (配合 Scratchpad)**：
    *   在 Hyprland 模式下，不要点击右上角的 X 关闭窗口（那会彻底退出进程）。
    *   **习惯养成**：看完邮件，再次按下 `$mainMod + M` 即可将其隐藏回后台。

---

## 3. 执行的指令

配置完成后，执行以下指令使更改生效：

```bash
# 1. 如果尚未安装，执行安装
sudo pacman -S thunderbird

# 2. 如果之前运行过 thunderbird，先彻底杀掉进程
killall thunderbird

# 3. 重新加载 Hyprland 配置 (或者直接使用快捷键 Super+Shift+C，取决于你的配置)
hyprctl reload

# 4. 手动启动一次测试 (或者直接重启电脑)
thunderbird &
```
