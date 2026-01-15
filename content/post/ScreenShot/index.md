---
title: "ScreenShot"
description: "用来配置hyprland的截屏"
date: 2025-12-25 21:52:25
tags:
   - Arch Linux
   - hyprland
categories:
   - Arch Linux
---

# Arch Linux Hyprland 终极截图配置指南 (Grim + Slurp + Satty + JQ)

本文档整理了构建一个具备“智能窗口吸附”、“拖拽选区”以及“现代化后期编辑”功能的截图工具链的完整过程。

## 1. 下载的包与依赖包

你需要安装基础截图工具、剪贴板工具、JSON处理工具（用于智能选区）以及编辑器。

**官方仓库 (Pacman):**
*   `grim`: Wayland 截图核心工具
*   `slurp`: 屏幕区域选择工具
*   `wl-clipboard`: Wayland 剪贴板工具 (`wl-copy`)
*   `jq`: 命令行 JSON 处理器 (实现单击窗口自动选中的关键)

**AUR (Yay/Paru):**
*   `satty`: 截图编辑器 (使用二进制版本避免编译耗时)

```bash
# 安装官方仓库依赖
sudo pacman -S grim slurp wl-clipboard jq

# 安装 Satty 编辑器
yay -S satty
```

---

## 2. 添加的配置信息

### 2.1 Hyprland 主配置
**文件位置:** `~/.config/hypr/hyprland.conf`

在配置文件中添加以下内容。这定义了变量、智能选区逻辑以及三个核心快捷键。

```ini
# ==========================================
# 截图工具链配置 (Grim + Slurp + Satty + JQ)
# ==========================================

# [变量] 定义截图默认存储目录
$screenshot_dir = $HOME/Pictures/Screenshots

# [核心逻辑] 定义“智能选区”命令
# 原理：hyprctl获取窗口信息 -> jq提取坐标并格式化 -> 传给slurp
# 效果：鼠标悬停窗口自动吸附(支持单击)，同时也支持按下左键拖拽选区
$smart_slurp = hyprctl clients -j | jq -r ".[] | \"\(.at[0]),\(.at[1]) \(.size[0])x\(.size[1])\"" | slurp

# -----------------------------------------------------

# [快捷键 1] Print 键 -> 智能截图 & 编辑 (日常最常用)
# 操作：按 Print -> 单击窗口 或 拖拽选区 -> 弹出 Satty 进行编辑 -> Ctrl+C 复制
bind = , Print, exec, bash -c '$smart_slurp | grim -g - - | satty --filename - --fullscreen --output-filename $screenshot_dir/Satty_$(date "+%Y%m%d-%H%M%S").png'

# [快捷键 2] Shift + Print -> 全屏截图 & 编辑
# 操作：按 Shift+Print -> 直接截取当前屏幕 -> 弹出 Satty 进行编辑
bind = SHIFT, Print, exec, bash -c 'grim - | satty --filename - --fullscreen --output-filename $screenshot_dir/Satty_$(date "+%Y%m%d-%H%M%S").png'

# [快捷键 3] Super + Print -> 快速后台截图
# 操作：按 Super+Print -> 智能选区 -> 自动保存文件并复制到剪贴板 (无弹窗) -> 发送通知
bind = SUPER, Print, exec, bash -c 'file=$screenshot_dir/Quick_$(date "+%Y%m%d-%H%M%S").png && $smart_slurp | grim -g - $file && wl-copy < $file && notify-send "截图已保存" "路径: $file"'
```

### 2.2 Satty 配置文件 (防崩溃版)
**文件位置:** `~/.config/satty/config.toml`

为了防止因版本不兼容导致的崩溃，使用以下最小化配置。

```toml
[general]
# 启动时默认全屏，方便编辑
fullscreen = true
# 截图后不立即退出 (根据个人喜好)
early-exit = false
```

---

## 3. 运行的指令

以下指令用于初始化环境、清理旧脚本以及使配置生效。

```bash
# 1. [文件系统] 创建截图保存目录
mkdir -p ~/Pictures/Screenshots

# 2. [配置清理] 删除之前创建的临时脚本文件 (现在逻辑已整合进 config，不再需要脚本)
rm ~/.config/hypr/scripts/screenshot.sh 2>/dev/null

# 3. [Satty修复] 确保 Satty 配置文件目录存在且配置正确
mkdir -p ~/.config/satty
# (如果你没有手动编辑上面的toml，可以用这行命令清空旧配置以防报错)
rm ~/.config/satty/config.toml 2>/dev/null

# 4. [Hyprland] 重新加载 Hyprland 配置以应用快捷键
hyprctl reload
```

---

## 4. 检查成功的步骤

请按照以下顺序进行测试，以确保所有功能正常工作。

### 测试 1: 智能选区与编辑 (Print)
1.  **操作**: 按下键盘上的 `Print Screen` (PrtSc) 键。
2.  **现象**: 鼠标光标变为十字。
3.  **交互 A (吸附)**: 移动鼠标到任意窗口上，该窗口应**自动高亮**。**单击**左键。
4.  **交互 B (拖拽)**: 或者在空白处按住左键**拖拽**一个矩形框。
5.  **结果**: 屏幕应弹出 **Satty** 编辑器窗口，显示你刚才选中的内容。
    *   尝试点击工具栏的裁剪 (Crop) 调整大小。
    *   尝试点击复制按钮或按 `Ctrl+C`。

### 测试 2: 全屏编辑 (Shift + Print)
1.  **操作**: 按下 `Shift + Print`。
2.  **结果**: 应直接弹出 **Satty** 窗口，内容为当前整个屏幕的截图。

### 测试 3: 快速截图 (Super + Print)
1.  **操作**: 按下 `Super (Windows键) + Print`。
2.  **操作**: 单击一个窗口或画一个框。
3.  **结果**:
    *   **没有** Satty 窗口弹出（静默模式）。
    *   系统应弹出一条通知 "截图已保存"。
    *   打开文件管理器查看 `~/Pictures/Screenshots` 目录下是否有新图片。
    *   找个聊天窗口按 `Ctrl+V`，应能粘贴出刚才的截图。
