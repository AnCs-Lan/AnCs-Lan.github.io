---
title: "其他配置"
description: "其他的一些小配置放在这里"
date: 2026-01-03 00:29:18
tags:
   - Arch Linux
categories:
   - Arch Linux
---

# 这里将放一些单独的小配置：


office文件食用餐具：**LibreOffice**
这是在 Hyprland (Wayland) 下体验最好的本地 Office 软件。它支持原生 Wayland 运行，不需要 XWayland 转译，因此画面清晰且输入法支持最好。
```zsh
sudo pacman -S libreoffice-fresh
```
**优化配置**:
为了让它在 Hyprland 下完美运行（使用 GTK 风格窗口边框），建议设置环境变量。编辑 ~/.config/hypr/hyprland.conf 或你的环境变量文件：
```Bash
env = SAL_USE_VCLPLUGIN,gtk3
```
**常见问题修复 (窗口规则)**:
如果你遇到弹窗（如“另存为”）被全屏拉伸的问题，可以在 hyprland.conf 中添加规则：
```ini
 windowrulev2 = float, class:(libreoffice), title:^(.*)(开启|Save|Open|Recovery)(.*)$
```

**office软件**:
```zsh
yay -S onlyoffice-bin
```
