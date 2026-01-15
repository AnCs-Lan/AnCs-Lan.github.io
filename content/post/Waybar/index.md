---
title: "Waybar"
description: "用waybar配置了一个还不错的任务栏"
date: 2025-12-25 19:52:46
tags:
   - "Arch Linux"
   - "hyprland"
categories:
   - "Arch Linux"
---

# Waybar "Cyber-Island" 配置部署指南

这份指南将帮助你部署刚才设计好的具有“科技感托盘抽屉”、“岛屿式模块分组”以及“红色警戒低电量特效”的 Waybar 配置。

## 1. 下载安装包与依赖

此配置基于 Arch Linux (Hyprland 环境)，请根据你的发行版调整包管理器命令。

### 核心组件
```bash
# 安装 Waybar (如果尚未安装)
sudo pacman -S waybar

# 安装 Nerd Fonts 字体 (必须，否则图标显示为乱码)
# 推荐 JetBrainsMono Nerd Font，配置中指定了该字体
sudo pacman -S ttf-jetbrains-mono-nerd
```

### 模块依赖工具
配置文件中调用了以下外部工具，需要安装才能让对应模块正常工作：

```bash
# 1. 音频可视化 (对应 config 中的 "cava")
sudo pacman -S cava

# 2. 屏幕亮度控制 (对应 config 中的 "backlight")
sudo pacman -S brightnessctl

# 3. 音量控制面板 (对应 config 中的 "on-click-right": "pwvucontrol")
# 如果没有 pwvucontrol，可以安装 pavucontrol 并修改配置，或者安装:
sudo pacman -S pwvucontrol

# 4. 网络管理界面 (对应 config 中的 "network")
sudo pacman -S iwgtk

# 5. 天气组件 (对应 config 中的 "custom/weather")
# 这是一个外部程序，通常在 AUR 中
yay -S wttrbar
```

---

## 2. 添加配置信息

请确保目录存在：`mkdir -p ~/.config/waybar`

### 配置文件: `~/.config/waybar/config`
*对应内容：完整的模块布局、左侧芯片抽屉定义、右侧硬件/电源分组定义。*

```json
[
  {
    "layer": "top",
    "position": "top",
    "name": "main",
    "height": 34,
    "reload_style_on_change": true,
    
    "margin-top": 6,
    "margin-left": 10,
    "margin-right": 10,
    
    "modules-left": [
      "group/tray-drawer",
      "hyprland/workspaces",
      "hyprland/window"
    ],

    "modules-center": [
      "clock"
    ],

    "modules-right": [
      "group/hardware",
      "custom/nvidia",
      "network",
      "group/audio",
      "cava",
      "custom/headsetbattery",
      "group/power",
      "custom/weather"
    ],

    // --- 左侧：科技感托盘抽屉 ---
    "group/tray-drawer": {
        "orientation": "horizontal",
        "drawer": {
            "transition-duration": 500,
            "children-class": "tray-child",
            "transition-left-to-right": true,
            "click-to-reveal": false
        },
        "modules": ["custom/tech-handle", "tray"]
    },

    "custom/tech-handle": {
        "format": "󰮫",
        "tooltip": false,
        "on-click": "bash ~/.config/scripts/pacman.sh" 
    },

    "tray": {
        "icon-size": 16,
        "spacing": 8
    },

    // --- 右侧：电源组 (电池+背光) ---
    "group/power": {
        "orientation": "horizontal",
        "drawer": {
            "transition-duration": 500,
            "children-class": "power-child",
            "transition-left-to-right": false
        },
        "modules": ["battery", "backlight"]
    },

    "battery": {
      "interval": 60,
      "states": {
          "warning": 30,
          "critical": 20
      },
      "format": "{icon} {capacity}%",
      "format-charging": "󰂄 {capacity}%",
      "format-plugged": " {capacity}%",
      "format-alt": "{time} {icon}",
      "format-full": "󰁹 {capacity}%",
      "format-icons": ["󰂎", "󰁺", "󰁻", "󰁼", "󰁽", "󰁾", "󰁿", "󰂀", "󰂁", "󰂂", "󰁹"],
      "tooltip-format": "{timeTo}\nPower: {power}W"
    },

    "backlight": {
      "device": "intel_backlight",
      "format": "{icon} {percent}%",
      "format-icons": ["󰃞", "󰃟", "󰃠"],
      "scroll-step": 5,
      "on-scroll-up": "brightnessctl set 2%-",
      "on-scroll-down": "brightnessctl set +2%"
    },

    // --- 硬件组 (CPU/Temp/RAM/Disk) ---
    "group/hardware": {
      "orientation": "horizontal",
      "drawer": {
        "transition-duration": 500,
        "children-class": "hw-child",
        "transition-left-to-right": false
      },
      "modules": ["cpu", "temperature", "memory", "disk"]
    },

    "cpu": {
      "interval": 10,
      "format": " {usage}%",
      "states": { "warning": 80, "critical": 95 }
    },
    
    "temperature": {
      "interval": 10,
      "format": "{icon} {temperatureC}°",
      "format-icons": ["", "", ""]
    },

    "memory": {
      "interval": 30,
      "format": " {percentage}%",
      "tooltip-format": "{used:0.1f}G/{total:0.1f}G"
    },

    "disk": {
      "interval": 30,
      "format": " {percentage_used}%",
      "path": "/"
    },

    // --- 音频组 ---
    "group/audio": {
      "orientation": "horizontal",
      "modules": ["pulseaudio#output", "pulseaudio#input"]
    },

    "pulseaudio#output": {
      "format": "{icon} {volume}%",
      "format-bluetooth": " {volume}%",
      "format-muted": " ",
      "format-icons": {
        "headphone": "",
        "hands-free": "",
        "headset": "",
        "phone": "",
        "portable": "",
        "car": "",
        "default": ["", "", ""]
      },
      "on-click": "pactl set-sink-mute @DEFAULT_SINK@ toggle",
      "on-click-right": "pwvucontrol",
      "scroll-step": 5
    },

    "pulseaudio#input": {
      "format": "{format_source}",
      "format-source": " {volume}%",
      "format-source-muted": " ",
      "on-click": "pactl set-source-mute @DEFAULT_SOURCE@ toggle",
      "scroll-step": 5
    },

    // --- 其他模块 ---
    
    "clock": {
      "interval": 1,
      "format": " {:%H:%M   %a %d.%m}",
      "tooltip-format": "<tt><small>{calendar}</small></tt>",
      "calendar": {
        "mode": "year",
        "mode-mon-col": 3,
        "weeks-pos": "right",
        "on-scroll": 1,
        "format": {
          "months": "<span color='#ffead3'><b>{}</b></span>",
          "days": "<span color='#ecc6d9'><b>{}</b></span>",
          "weeks": "<span color='#99ffdd'><b>W{}</b></span>",
          "weekdays": "<span color='#ffcc66'><b>{}</b></span>",
          "today": "<span color='#ff6699'><b><u>{}</u></b></span>"
        }
      },
      "on-click": "chromium --app=https://calendar.google.com & aplay ~/.config/sounds/interact.wav"
    },

    "hyprland/workspaces": {
      "disable-scroll-wraparound": true,
      "all-outputs": true,
      "format": "{icon}",
      "format-icons": {
        "1": "一",
        "2": "二",
        "3": "三",
        "4": "四",
        "5": "五",
        "6": "六",
        "7": "七",
        "8": "八",
        "9": "九",
        "10": "十",
        "default": ""
      }
    },

    "hyprland/window": {
      "format": "{title}",
      "max-length": 30,
      "separate-outputs": true,
      "icon": true,
      "icon-size": 16
    },

    "custom/nvidia": {
      "interval": 10,
      "return-type": "json",
      "exec": "~/.config/hypr/themes/Mocha-Power/nvidia.sh",
      "format": "󰢮 {text}",
    },

    "network": {
      "interval": 5,
      "format-wifi": " {essid}",
      "format-ethernet": "󰈀 Wired",
      "format-linked": "󰈀 {ifname} (No IP)",
      "format-disconnected": "󰤮 Disconnected",
      "tooltip-format": "{ifname} via {gwaddr} 󰇚{bandwidthDownBits} 󰕒{bandwidthUpBits}",
      "on-click": "iwgtk & aplay ~/.config/sounds/interact.wav"
    },

    "custom/headsetbattery": {
      "interval": 60,
      "return-type": "json",
      "format": " {}",
      "exec": "~/.config/hypr/themes/Mocha-Power/headsetbattery.sh"
    },

    "custom/weather": {
      "format": "{}",
      "interval": 300,
      "exec": "wttrbar --nerd --location Oulu",
      "return-type": "json"
    }
  }
]
```

### 样式表: `~/.config/waybar/style.css`
*对应内容：半透明岛屿风格、左侧芯片霓虹特效、电池红色警戒特效。*

```css
/* 全局设定 */
* {
    font-family: "JetBrainsMono Nerd Font", FontAwesome, Roboto, Helvetica, Arial, sans-serif;
    font-size: 14px;
    font-weight: bold;
}

/* 窗口透明 */
window#waybar {
    background-color: rgba(0, 0, 0, 0.0);
    color: #cdd6f4;
    transition-property: background-color;
    transition-duration: .5s;
}

/* =========================================
   左侧：科技感托盘抽屉 (Tech Drawer)
   ========================================= */

#group-tray-drawer {
    background-color: rgba(20, 20, 30, 0.95);
    border: 1px solid rgba(137, 180, 250, 0.3);
    border-radius: 12px;
    margin: 4px; 
    padding: 2px;
    transition: all 0.3s ease-in-out;
}

#group-tray-drawer:hover {
    box-shadow: 0 0 10px rgba(137, 180, 250, 0.3);
    border-color: rgba(137, 180, 250, 0.8);
}

/* 把手符号 (芯片)：霓虹呼吸特效 */
#custom-tech-handle {
    color: #89b4fa;
    font-size: 20px;
    padding: 0 8px;
    margin: 0;
    background-color: transparent;
    text-shadow: 0 0 5px rgba(137, 180, 250, 0.6),
                 0 0 10px rgba(137, 180, 250, 0.3);
    animation: tech-pulse 4s infinite cubic-bezier(0.4, 0, 0.6, 1);
}

#tray {
    background-color: transparent; 
    margin: 0;
    padding: 0 8px;
}
#tray > .passive { -gtk-icon-effect: dim; }
#tray > .needs-attention { -gtk-icon-effect: highlight; background-color: #f38ba8; }

/* =========================================
   右侧：岛屿风格模块
   ========================================= */

#workspaces,
#clock,
#network,
#custom-weather,
#custom-nvidia,
#custom-headsetbattery,
#group-hardware,
#group-audio,
#group-power, 
#cava {
    background-color: rgba(30, 30, 46, 0.85); 
    padding: 4px 12px;
    margin: 4px;
    border-radius: 12px;
    border: 1px solid rgba(180, 190, 254, 0.1);
    box-shadow: 1px 1px 3px rgba(0,0,0,0.3);
}

#group-hardware > widget > *,
#group-audio > widget > *,
#group-power > widget > * {
    background-color: transparent;
    margin: 0;
    border: none;
    box-shadow: none;
}

/* =========================================
   特定模块微调
   ========================================= */

#workspaces button {
    padding: 0 5px;
    background-color: transparent;
    color: #bac2de;
    border-radius: 8px;
}
#workspaces button:hover { background-color: rgba(180, 190, 254, 0.2); }
#workspaces button.active { color: #a6e3a1; background-color: rgba(166, 227, 161, 0.1); }
#workspaces button.urgent { color: #f38ba8; }

#clock { color: #f9e2af; }
#network { color: #89b4fa; }
#custom-weather { color: #89dceb; }

/* =========================================
   电池特效 (Battery)
   ========================================= */

#battery { color: #a6e3a1; }
#battery.charging, #battery.plugged { color: #94e2d5; }
#battery.warning:not(.charging) { color: #fab387; }

/* 极低电量红色警报 (Critical) */
#battery.critical:not(.charging) {
    color: #f38ba8; 
    background-color: transparent; 
    animation: critical-pulse 1s infinite alternate cubic-bezier(0.4, 0, 0.2, 1);
}

/* =========================================
   动画定义
   ========================================= */

@keyframes tech-pulse {
    0% { opacity: 0.7; text-shadow: 0 0 3px rgba(137, 180, 250, 0.4); }
    50% { opacity: 1; text-shadow: 0 0 10px rgba(137, 180, 250, 0.9); }
    100% { opacity: 0.7; text-shadow: 0 0 3px rgba(137, 180, 250, 0.4); }
}

@keyframes critical-pulse {
    0% {
        border-color: rgba(243, 139, 168, 0.3);
        text-shadow: 0 0 0 rgba(243, 139, 168, 0);
        box-shadow: 0 0 0 rgba(243, 139, 168, 0);
    }
    100% {
        border-color: rgba(243, 139, 168, 1);
        text-shadow: 0 0 10px rgba(243, 139, 168, 0.8);
        box-shadow: 0 0 15px rgba(243, 139, 168, 0.4);
    }
}
```

---

## 3. 运行指令

### 赋予自定义脚本权限
你的配置中引用了一些自定义脚本 (`exec` 字段)，**必须确保它们存在并有执行权限**，否则对应的模块（Nvidia, HeadsetBattery, Launcher）将显示为空或报错。

```bash
# 赋予权限（请根据你实际的脚本路径运行）
chmod +x ~/.config/hypr/themes/Mocha-Power/nvidia.sh
chmod +x ~/.config/hypr/themes/Mocha-Power/headsetbattery.sh
chmod +x ~/.config/scripts/pacman.sh
```

### 启动 Waybar
在终端中运行以下命令来启动（或重启）Waybar，以便查看实时日志（用于除错）：

```bash
# 终止正在运行的实例并重新启动
killall waybar && waybar
```
*对应配置：此命令会读取 `~/.config/waybar/config` 和 `~/.config/waybar/style.css`。*

---

## 4. 检查成功的步骤

请按以下顺序检查你的新配置是否生效：

1.  **左侧抽屉检查**:
    *   **现象**: 左上角应该只看到一个蓝色的、带有呼吸灯效的芯片符号 (`󰮫`)。
    *   **动作**: 将鼠标悬停在芯片上。
    *   **预期**: 芯片右侧应平滑滑出，显示系统托盘图标（如网络、输入法）。鼠标移开后自动收回。

2.  **右侧岛屿分组检查**:
    *   **现象**: 右侧的模块（如 CPU/温度, 音量, 电池）应该被包裹在深色半透明圆角背景中，而不是散落在桌面上。
    *   **动作**: 观察 `group/power` 组。
    *   **预期**: 电池图标和屏幕亮度图标应该并排显示在同一个背景“岛屿”中。

3.  **电池警报特效检查 (可选)**:
    *   **动作**: 拔掉电源，如果你能将电量消耗到 20% 以下（或暂时修改 config 中的 critical 阈值为比当前电量高的数值）。
    *   **预期**: 电池文字变红，并且整个电池模块边缘出现**红色的呼吸光晕**（Red Alert），而不是变成一个红色的实心方块。

4.  **字体检查**:
    *   **现象**: 所有的图标（如 , 󰂄, ）都应该清晰显示。
    *   **故障排除**: 如果显示为方框 `[]` 或乱码，说明 `ttf-jetbrains-mono-nerd` 字体未正确安装或未生效。
