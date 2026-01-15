---
title: "Steam&GPUChange"
descriprtion: "配置了一下linux的steam,同时把混显的切换配置了一下"
date: 2026-01-11 17:01:15
tags:
   - "Arch Linux"
   - "hyprland"
categories:
   - "Arch Linux"
---

# Arch Linux + Hyprland + NVIDIA RTX 5070 配置指南

本指南基于 **EnvyControl** 进行显卡模式切换（核显模式/独显模式），并修复 Steam 在 Wayland 下的黑屏问题。

## 1. 安装的包以及依赖包

首先确保 `/etc/pacman.conf` 中 `[multilib]` 仓库已开启。

### 核心驱动与工具
```bash
# 基础 NVIDIA 驱动与相关工具
sudo pacman -S nvidia nvidia-utils nvidia-settings nvidia-prime

# 32位驱动支持 (Steam 运行的核心依赖，缺少会导致无法启动或黑屏)
sudo pacman -S lib32-nvidia-utils lib32-mesa lib32-vulkan-intel vulkan-intel

# 显卡切换工具 (需通过 AUR 安装，假设使用 yay 或 paru)
yay -S envycontrol
```

### 软件与字体
```bash
# Steam 本体与防乱码字体
sudo pacman -S steam ttf-liberation
```

---

## 2. 修改的配置

### A. 内核启动参数 (`/etc/default/grub`)
为了让 NVIDIA 驱动在 Wayland 下正常加载。

找到 `GRUB_CMDLINE_LINUX_DEFAULT` 行，**追加**以下参数（保留原有内容）：
```bash
GRUB_CMDLINE_LINUX_DEFAULT="...你的原有参数... nvidia_drm.modeset=1"
```
*注意：对于混合显卡环境，建议暂时不要加 `nvidia_drm.fbdev=1`，以防冲突。*

### B. 初始化内存盘 (`/etc/mkinitcpio.conf`)
解决启动黑屏问题，确保驱动在内核早期加载。

找到 `MODULES=()` 行，修改为：
```bash
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```

### C. Hyprland 环境变量 (`~/.config/hypr/hyprland.conf`)
优化 Wayland 体验并适配 NVIDIA。

```ini
# --- 基础外观与应用兼容 ---
env = XCURSOR_SIZE,24
env = HYPRCURSOR_SIZE,24
env = HYPRLAND_FONT,JetBrainsMono Nerd Font:size=10
env = EDITOR,nvim
env = VISUAL,nvim

# 强制应用使用 Wayland 后端 (修复画面模糊)
env = GDK_BACKEND,wayland,x11,*
env = QT_QPA_PLATFORM,wayland;xcb
env = SDL_VIDEODRIVER,wayland
env = CLUTTER_BACKEND,wayland

# Electron/Steam 修复
env = ELECTRON_OZONE_PLATFORM_HINT,auto

# --- 显卡相关配置 (需根据模式调整) ---

# 【情况 1：使用 integrated (纯核显) 或 hybrid (混合) 模式时】
# 请保留下方 NVIDIA 相关变量为 注释状态 (#)，否则可能导致黑屏或无法进入桌面。

# 【情况 2：使用 nvidia (纯独显) 模式时】
# 请 取消注释 下方变量：
# env = LIBVA_DRIVER_NAME,nvidia
# env = XDG_SESSION_TYPE,wayland
# env = GBM_BACKEND,nvidia-drm
# env = __GLX_VENDOR_LIBRARY_NAME,nvidia
# env = NVD_BACKEND,direct

# --- 鼠标光标修复 (写在配置文件的常规区域，非 env) ---
cursor {
    no_hardware_cursors = true
}
```

---

## 3. 执行的指令

### 应用系统配置 (修改配置文件后必须执行)
```bash
# 1. 重新生成 initramfs 镜像
sudo mkinitcpio -P

# 2. 更新 GRUB 引导配置
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### 显卡模式切换 (使用 EnvyControl)
```bash
# 切换到 纯独显模式 (适合插电玩大作，需配合修改 Hyprland env)
sudo envycontrol -s nvidia

# 切换到 纯核显模式 (适合外出办公，最稳定，需注释掉 Hyprland env 中的 nvidia 变量)
sudo envycontrol -s integrated

# 切换到 混合模式 (默认)
sudo envycontrol -s hybrid

# 切换后需重启电脑
reboot
```

### 修复 Steam 黑屏/无法启动 (仅在出问题时执行)
如果进入 Steam 界面黑屏，请按顺序执行：
1.  **清理缓存**：
    ```bash
    rm -rf ~/.local/share/Steam/appcache/httpcache
    rm -rf ~/.local/share/Steam/config/htmlcache
    ```
2.  **临时禁用 GPU 加速启动**：
    ```bash
    steam -cef-disable-gpu
    ```
3.  **永久设置**：
    进入 Steam 设置 -> 界面 -> **取消勾选**“在网页视图中启用 GPU 加速渲染”。

---

## 4. 检查成功的方式

### 1. 检查内核参数是否生效
```bash
cat /proc/cmdline
```
*输出中应包含 `nvidia_drm.modeset=1`。*

### 2. 检查当前显卡模式
```bash
sudo envycontrol -q
```
*显示当前的模式（integrated / hybrid / nvidia）。*

### 3. 验证 NVIDIA 驱动状态 (仅在 Nvidia/Hybrid 模式下)
```bash
nvidia-smi
```
*应显示 RTX 5070 的详细信息和驱动版本。*

### 4. 验证游戏是否运行在独显上
在游戏运行时（如果在 Hybrid 模式下需加启动参数 `prime-run %command%`），在终端运行：
```bash
nvtop
```
*应该能看到对应的游戏进程占用 NVIDIA GPU 资源。*


###附：解决反作弊游戏无法运行的问题：
在游戏的属性，通用，启动选项加入这样的一句话：
```
env -u SDL_VIDEODRIVER %command%
```

### 给你的懒人优化建议

每次切换显卡都要去改配置文件太麻烦了。你可以把这几行单独拆出来，做成两个小文件：

1.  在 `~/.config/hypr/` 目录下新建一个文件叫 `nvidia.conf`，里面写：
    ```ini
    env = LIBVA_DRIVER_NAME,nvidia
    env = XDG_SESSION_TYPE,wayland
    env = GBM_BACKEND,nvidia-drm
    env = __GLX_VENDOR_LIBRARY_NAME,nvidia
    env = NVD_BACKEND,direct
    ```

2.  再建一个文件叫 `intel.conf`，里面**什么都不写**（或者是空的）。

3.  修改你的主配置文件 `hyprland.conf`，在原来写这些变量的地方，改成一句引用：
    ```ini
    # 加载显卡配置
    source = ~/.config/hypr/gpu.conf
    ```

4.  **切换的时候怎么做？**
    *   想用**独显模式**时，在终端运行：`cp ~/.config/hypr/nvidia.conf ~/.config/hypr/gpu.conf`
    *   想用**核显/混合模式**时，在终端运行：`cp ~/.config/hypr/intel.conf ~/.config/hypr/gpu.conf`

这样你配合 `envycontrol` 切换显卡时，只需要顺手复制一下文件，不用每次进去改代码，不容易出错。
