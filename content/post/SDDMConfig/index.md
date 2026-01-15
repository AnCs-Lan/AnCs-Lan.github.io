---
title: "SDDM Config"
description: "配置了一个SDDM启动页面来代替tty启动"
date: 2025-12-24 19:07:04
tags:
   - Arch Linux
   - hyprland
categories:
   - Arch Linux
---

# Arch Linux SDDM 配置指南 (Catppuccin + Sugar Candy)

本指南专注于 SDDM 的安装、主题配置与美化。

## 1. 下载的包、依赖包

**模块说明**：包含显示管理器本体、Qt5 图形渲染库（主题必需）、Sugar Candy 主题、字体及密钥环组件。

```bash
# 1. 安装基础组件、Qt5 依赖、字体、密钥环
sudo pacman -S sddm qt5-graphicaleffects qt5-quickcontrols2 qt5-svg \
ttf-jetbrains-mono-nerd gnome-keyring libsecret

# 2. 安装 Sugar Candy 主题 (通过 AUR)
yay -S sddm-sugar-candy-git
```

---

## 2. 添加的配置信息

**模块说明**：涉及主服务配置、主题外观定制以及 PAM 权限配置。

### A. SDDM 主配置文件
**文件路径**: `/etc/sddm.conf`
**功能**: 指定启用 Sugar Candy 主题，并禁用虚拟键盘。

```ini
[General]
# 禁用虚拟键盘
InputMethod=

[Theme]
# 必须与 /usr/share/sddm/themes/ 下的文件夹名称一致
Current=sugar-candy
ThemeDir=/usr/share/sddm/themes
```

### B. 主题深度定制 (Catppuccin Mocha 配色 + 居中布局)
**文件路径**: `/usr/share/sddm/themes/sugar-candy/theme.conf`
**功能**: 定义壁纸、配色方案、圆角及 UI 布局。

```ini
[General]

Background="Backgrounds/NvWuLing.jpg"
DimBackgroundImage="0.0"
ScaleImageCropped="true"
ScreenWidth="2100"
ScreenHeight="1680"

## [Blur Settings]

FullBlur="false"
PartialBlur="true"
BlurRadius="0"

## [Design Customizations]

HaveFormBackground="false"
FormPosition="center"
BackgroundImageHAlignment="center"
BackgroundImageVAlignment="center"
# ------------------------------------------------
# MainColor -> Catppuccin "Text" (#cdd6f4)
# 这是未被选中时的图标和文字颜色
# ------------------------------------------------
MainColor="#cdd6f4"
# ------------------------------------------------
# AccentColor -> Catppuccin "Mauve" (#cba6f7) 
# 或者是 "Blue" (#89b4fa)
# 这是选中框、光标、登录按钮背景色。这是你系统的主色调。
# ------------------------------------------------
AccentColor="#cba6f7"
# ------------------------------------------------
# BackgroundColor -> Catppuccin "Base" (#1e1e2e)
# 这是输入框那一块矩形的背景色（如果你开启了 FormBackground）
# ------------------------------------------------
BackgroundColor="#1e1e2e"
# ------------------------------------------------
# OverrideLoginButtonTextColor -> Catppuccin "Base" (#1e1e2e)
# 这一步很关键！
# 默认登录按钮文字也是 MainColor (浅色)，
# 但如果你的 AccentColor 也是浅色 (如淡紫色)，
# "浅色字+浅色按钮" 会看不清。
# 所以强制把按钮文字改成深色背景色，对比度才够高。
# ------------------------------------------------
OverrideLoginButtonTextColor="#1e1e2e"
InterfaceShadowSize="6"
InterfaceShadowOpacity="0.6"
RoundCorners="20"
ScreenPadding="0"
Font="JetBrainsMono Nerd Font"
FontSize=""

## [Interface Behavior]

ForceRightToLeft="false"
ForceLastUser="true"
ForcePasswordFocus="true"
ForceHideCompletePassword="false"
ForceHideVirtualKeyboardButton="false"
ForceHideSystemButtons="false"
AllowEmptyPassword="false"
AllowBadUsernames="false"

## [Locale Settings]

Locale=""
HourFormat="HH:mm"
DateFormat="dddd, d of MMMM"

## [Translations]

HeaderText="Welcome Back, My Friend"
TranslatePlaceholderUsername=""
TranslatePlaceholderPassword=""
TranslateShowPassword=""
TranslateLogin=""
TranslateLoginFailedWarning=""
TranslateCapslockWarning=""
TranslateSession=""
TranslateSuspend=""
TranslateHibernate=""
TranslateReboot=""
TranslateShutdown=""
TranslateVirtualKeyboardButton=""
```

### C. PAM 权限配置 (解决 Keyring 解锁)
**文件路径**: `/etc/pam.d/sddm`
**功能**: 登录时自动解锁 gnome-keyring，避免进系统后重复输密码。

```pam
#%PAM-1.0
auth     include       system-login
# 添加此行：
auth     optional      pam_gnome_keyring.so

account  include       system-login
password include       system-login

session  include       system-login
# 添加此行：
session  optional      pam_gnome_keyring.so auto_start
```

---

## 3. 运行的指令

**模块说明**：文件操作、权限修复与服务管理。

```bash
# [对应配置 B] 放置壁纸
# 将你的壁纸复制到主题目录，并重命名为 theme.conf 中对应的名字 (如 wall.jpg)
sudo cp ~/Pictures/your_wallpaper.jpg /usr/share/sddm/themes/sugar-candy/Backgrounds/wall.jpg

# [对应配置 B] 修复主题权限
# 确保 SDDM 用户有权限读取主题文件，防止回退默认主题
sudo chown -R root:root /usr/share/sddm/themes/sugar-candy
sudo chmod -R 755 /usr/share/sddm/themes/sugar-candy

# [对应配置 A] 清理干扰并启用服务
# 删除可能存在的旧配置干扰
sudo rm -rf /etc/sddm.conf.d/*
# 启用 SDDM 开机自启
sudo systemctl enable sddm
# 确保系统默认进入图形界面
sudo systemctl set-default graphical.target
```

---

## 4. 检查成功的步骤

**模块说明**：在重启前验证配置是否生效。

1.  **执行测试命令**：
    在终端输入（不需要 sudo）：
    ```bash
    sddm-greeter --test-mode
    ```

2.  **验证标准**：
    *   **日志检查**：终端输出中必须包含 `Loading theme configuration from "/usr/share/sddm/themes/sugar-candy/theme.conf"`。
    *   **视觉检查**：弹出的窗口背景是你设置的壁纸，输入框居中，颜色为 Catppuccin 风格。

3.  **完成**：
    验证无误后，执行 `reboot` 重启系统。
