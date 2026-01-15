---
title: "RofiConfig"
description: "用Rofi配置了一个蓝色的启动器"
date: 2025-12-25 19:52:17
tags:
   - Arch Linux
   - hyprland
categories:
   - Arch Linux
---

# Arch Linux + Hyprland: Rofi 极简淡蓝半透明侧边栏配置指南

这份指南包括：淡蓝色半透明风格、侧边栏布局、去除斑马纹色块、修复图标遮挡以及 Hyprland 的毛玻璃特效设置。

---

## 1. 下载包与依赖

我们需要安装 `rofi-wayland`（启动器）、剪切板工具以及字体（用于显示图标）。

```bash
# 安装核心组件和字体
sudo pacman -S rofi-wayland wl-clipboard cliphist ttf-jetbrains-mono-nerd

# (可选) 如果你需要图标主题，推荐 Papirus
sudo pacman -S papirus-icon-theme
```

---

## 2. 添加配置信息

我们需要修改两个文件：Hyprland 的配置文件（负责运行后台服务、绑定快捷键和开启模糊特效）和 Rofi 的配置文件（负责界面样式）。

### A. 修改 Hyprland 配置
**文件位置：** `~/.config/hypr/hyprland.conf`

在文件中添加或修改以下内容：

```ini
# -----------------------------------------------------
# 1. 自启动剪切板守护进程
# -----------------------------------------------------
exec-once = wl-paste --type text --watch cliphist store
exec-once = wl-paste --type image --watch cliphist store

# -----------------------------------------------------
# 2. 开启 Rofi 的毛玻璃 (Blur) 特效
# -----------------------------------------------------
# 告诉 Hyprland 模糊 Rofi 的背景，必须要加，否则半透明背景会看不清
layerrule = blur, rofi
layerrule = ignorezero, rofi

# -----------------------------------------------------
# 3. 绑定快捷键
# -----------------------------------------------------
# Super + A 打开应用启动器
bind = $mainMod, A, exec, rofi -show drun

# Super + V 打开剪切板历史
# 注意：这里复用了 Rofi 的样式，只不过把图标临时换成了 Clipboard 文字
bind = $mainMod, V, exec, cliphist list | rofi -dmenu -p "Clipboard" -display-columns 2 | cliphist decode | wl-copy
```

### B. 修改 Rofi 样式配置 (最终完美版)
**文件位置：** `~/.config/rofi/config.rasi`

如果文件不存在则新建，将以下内容**完全覆盖**原有内容：

```css
/*******************************************************************************
 * Rofi 最终完美版 - 淡蓝半透明侧边栏风格
 *******************************************************************************/

configuration {
    modi:                       "drun,run,window";
    show-icons:                 true;
    /* 侧边栏顶部的大图标定义在这里 */
    display-drun:               "";
    display-run:                "";
    display-window:             "";
    drun-display-format:        "{name}";
    window-format:              "{w} · {c} · {t}";
    /* 全局字体 */
    font:                       "JetBrainsMono Nerd Font Bold 10";
}

/* --- 全局变量与配色 --- */
* {
    /* 背景色: 深蓝灰 + 90% 透明度 (E6) */
    background:                 #1E2127E6;
    
    /* 字体颜色 */
    foreground:                 #D8DEE9;
    
    /* 选中高亮色: 冰蓝色 */
    selected:                   #89DCEB;
    selected-text:              #1E2127;
    
    /* 边框颜色 */
    border-col:                 #89DCEB;
    
    /* 左侧面板的渐变背景 */
    sidebar-gradient:           linear-gradient(to bottom, #2E3440, #1E2127);
    
    /* 关键：强制全局背景透明，防止白色色块出现 */
    background-color:           transparent;
    text-color:                 @foreground;
}

/* --- 主窗口容器 --- */
window {
    transparency:               "real";
    location:                   center;
    anchor:                     center;
    fullscreen:                 false;
    width:                      800px;
    height:                     500px;
    enabled:                    true;
    border:                     2px;
    border-color:               @border-col;
    border-radius:              16px;
    background-color:           @background;
    cursor:                     "default";
}

/* --- 主布局：水平排列 (左侧栏 | 右侧列表) --- */
mainbox {
    enabled:                    true;
    orientation:                horizontal;
    children:                   [ "box-left", "listview" ];
    background-color:           transparent;
}

/* --- 左侧面板 --- */
box-left {
    enabled:                    true;
    width:                      280px;
    background-image:           @sidebar-gradient;
    orientation:                vertical;
    children:                   [ "inputbar", "dummy-spacer", "mode-switcher" ];
    padding:                    30px;
    spacing:                    20px;
}

/* 左侧输入栏区域 */
inputbar {
    enabled:                    true;
    orientation:                vertical;
    spacing:                    20px;
    background-color:           transparent;
    children:                   [ "prompt", "entry" ];
}

/* 大图标 (由 Prompt 伪装) */
prompt {
    enabled:                    true;
    /* 修复遮挡：字体调小至 65，增加垂直 padding */
    font:                       "JetBrainsMono Nerd Font Bold 65";
    padding:                    20px 0px; 
    background-color:           transparent;
    text-color:                 @selected;
    horizontal-align:           0.5;
    vertical-align:             0.5;
}

/* 搜索输入框 */
entry {
    enabled:                    true;
    padding:                    12px;
    border-radius:              10px;
    background-color:           #3B4252;
    text-color:                 @foreground;
    cursor:                     text;
    placeholder:                "Welcome User"; /* 这里是问候语 */
    placeholder-color:          #8fa1b3;
    horizontal-align:           0.5;
}

/* 占位符 (把按钮顶到底部) */
dummy-spacer {
    expand:                     true;
    background-color:           transparent;
}

/* 底部模式切换按钮 */
mode-switcher {
    enabled:                    true;
    spacing:                    10px;
    background-color:           transparent;
}

button {
    padding:                    10px;
    border-radius:              8px;
    background-color:           #3B4252;
    text-color:                 inherit;
    cursor:                     pointer;
}

button selected {
    background-color:           @selected;
    text-color:                 @selected-text;
}

/* --- 右侧列表区域 --- */
listview {
    enabled:                    true;
    columns:                    1;
    lines:                      8;
    padding:                    30px;
    spacing:                    10px;
    background-color:           transparent;
}

/* 列表项基础样式 */
element {
    enabled:                    true;
    spacing:                    15px;
    padding:                    12px 15px;
    border-radius:              10px;
    cursor:                     pointer;
    background-color:           transparent;
    text-color:                 @foreground;
}

/* === 状态样式定义 (彻底修复白色色块) === */

/* 普通状态 */
element normal.normal {
    background-color:           transparent;
    text-color:                 @foreground;
}
/* 交替行状态 */
element alternate.normal {
    background-color:           transparent;
    text-color:                 @foreground;
}
/* 已打开窗口状态 */
element normal.active {
    background-color:           transparent;
    text-color:                 @selected;
}
element alternate.active {
    background-color:           transparent;
    text-color:                 @selected;
}

/* 选中状态 (高亮) */
element selected.normal {
    background-color:           @selected;
    text-color:                 @selected-text;
}
element selected.active {
    background-color:           @selected;
    text-color:                 @selected-text;
}

/* 图标与文字 */
element-icon {
    background-color:           transparent;
    text-color:                 inherit;
    size:                       26px;
    cursor:                     inherit;
}
element-text {
    background-color:           transparent;
    text-color:                 inherit;
    cursor:                     inherit;
    vertical-align:             0.5;
}
```

---

## 3. 运行指令

在修改完上述配置后，你需要通过终端测试或重启 Hyprland 来生效。

**1. 重载 Hyprland 配置 (使快捷键和模糊特效生效):**
通常直接在终端输入无法生效，建议直接使用快捷键退出 Hyprland 重新登录，或者使用默认快捷键（通常是 `Super + M` 或 `Super + Shift + R`，视你的配置而定）重载。

**2. 测试应用启动器 (对应 Rofi Drun 模块):**
```bash
rofi -show drun
```
*如果你已经绑定了快捷键，直接按 `Super + A`。*

**3. 测试剪切板 (对应 Cliphist + Rofi Dmenu 模块):**
```bash
cliphist list | rofi -dmenu -p "Clipboard" | cliphist decode | wl-copy
```
*如果你已经绑定了快捷键，直接按 `Super + V`。*

---

## 4. 检查成功的步骤

1.  **功能检查**：
    *   按下 `Super + A`，是否弹出了淡蓝色窗口？
    *   输入应用名称（例如 `firefox`），是否能搜索并启动？
    *   按下 `Super + V`，是否显示了之前的复制历史？

2.  **视觉检查**：
    *   **左侧栏**：是否看到了巨大的 Arch Logo () 且**没有被遮挡**？下方是否有 "Welcome User" 字样？
    *   **背景**：窗口背景是否是半透明磨砂的（能隐约看到后面的壁纸）？
    *   **列表**：右侧列表在未选中时是否完全透明（没有白色或灰色的条纹块）？选中时是否变成冰蓝色圆角条？

如果以上检查全部通过，恭喜你，你已经拥有了一套颜值极高的 Arch Linux 应用管理界面！
