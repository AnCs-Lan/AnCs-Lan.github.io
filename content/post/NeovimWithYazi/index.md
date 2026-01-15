---
title: "Neovim + Yazi"
decription: "一份把lazyvim和yazi联合配置的文档"
date: 2025-12-25 19:53:23
tags:
   - Arch Linux
categories:
   - Arch Linux
---
# Arch Linux Neovim (LazyVim) + Yazi + Kitty 终极配置指南

本文档旨在将 Arch Linux 上的 Neovim 打造成具备 IDE 功能（代码补全、文件树、调试）、现代化 UI（透明磨砂、光标特效）的环境，并实现与 Yazi 文件管理器的双向无缝集成。

---

## 1. 安装包与依赖

首先安装系统基础工具、编辑器、文件管理器及必要的字体和终端。

```bash
# 1. 基础构建工具 (编译插件如 telescope-fzf-native 需用到)
sudo pacman -S base-devel cmake unzip ninja curl

# 2. Neovim 核心组件
# - neovim: 编辑器
# - ripgrep: 极速搜索 (Telescope/Snacks 依赖)
# - fd: 极速文件查找 (Telescope/Snacks 依赖)
# - npm: Node.js 包管理器 (部分 LSP 服务端依赖)
sudo pacman -S neovim ripgrep fd npm

# 3. Yazi 及预览依赖
# - yazi: 文件管理器
# - ffmpegthumbnailer: 视频预览
# - p7zip, jq, poppler, imagemagick: 各类文件预览支持
# - zoxide: 智能目录跳转
sudo pacman -S yazi ffmpegthumbnailer p7zip jq poppler imagemagick zoxide

# 4. 字体与剪切板 (IDE 体验必需)
# - ttf-jetbrains-mono-nerd: 解决图标乱码
# - wl-clipboard: Wayland 剪切板支持 (X11 用户请装 xclip)
sudo pacman -S ttf-jetbrains-mono-nerd wl-clipboard

# 5. 终端模拟器 (推荐 Kitty)
sudo pacman -S kitty
```

---

## 2. 配置文件与内容

### 2.1 初始化 LazyVim
如果尚未配置 Neovim，请先初始化 LazyVim 模板：

```bash
# 备份旧配置
mv ~/.config/nvim{,.bak}
mv ~/.local/share/nvim{,.bak}

# 克隆模板
git clone https://github.com/LazyVim/starter ~/.config/nvim

# 移除 git 关联
rm -rf ~/.config/nvim/.git
```

### 2.2 Neovim 插件配置
以下文件位于 `~/.config/nvim/lua/plugins/` 目录下。

#### A. `lua/plugins/yazi.lua` (Yazi 集成)
**功能**：在 Neovim 内部通过快捷键呼出 Yazi 悬浮窗，选中文件后在当前 Buffer 打开。

```lua
return {
  {
    "mikavilpas/yazi.nvim",
    event = "VeryLazy",
    keys = {
      -- [关联] 快捷键：空格 + 减号 -> 打开 Yazi
      {
        "<leader>-",
        "<cmd>Yazi<cr>",
        desc = "Open Yazi (File Manager)",
      },
      -- [关联] 快捷键：空格 + cw -> 在当前文件目录打开 Yazi
      {
        "<leader>cw",
        "<cmd>Yazi cwd<cr>",
        desc = "Open Yazi in current working directory",
      },
    },
    opts = {
      open_for_directories = false,
      keymaps = {
        show_help = '<f1>',
      },
    },
  },
  -- 可选：禁用 LazyVim 默认的 Neo-tree，完全改用 Yazi
  {
    "nvim-neo-tree/neo-tree.nvim",
    enabled = false,
  },
}
```

#### B. `lua/plugins/theme.lua` (主题与透明化)
**功能**：设置 Catppuccin 主题，开启全局透明，并修复浮窗（如搜索框）不透明的问题。

```lua
return {
  {
    "catppuccin/nvim",
    name = "catppuccin",
    priority = 1000,
    opts = {
      flavour = "mocha", -- 风格: 深色
      transparent_background = true, -- [关联] 开启 Neovim 透明背景
      integrations = {
        cmp = true,
        gitsigns = true,
        nvimtree = true,
        treesitter = true,
        mini = { enabled = true },
        snacks = true, -- 新版 LazyVim 支持
      },
      -- [关联] 关键：强制去除悬浮窗背景色，配合终端透明
      custom_highlights = function(colors)
        return {
          NormalFloat = { bg = colors.none },
          FloatBorder = { bg = colors.none },
          SnacksPickerNormal = { bg = colors.none },
          SnacksPickerBorder = { bg = colors.none },
          SnacksPickerTitle = { bg = colors.none },
          SnacksPickerInput = { bg = colors.none },
          TelescopeNormal = { bg = colors.none },
          TelescopeBorder = { bg = colors.none },
        }
      end,
    },
  },
  -- 激活主题
  {
    "LazyVim/LazyVim",
    opts = {
      colorscheme = "catppuccin",
    },
  },
}
```

#### C. `lua/plugins/flatten.lua` (防嵌套)
**功能**：防止在 Neovim 的终端里再次打开 Neovim 时出现“套娃”现象。

```lua
return {
  {
    "willothy/flatten.nvim",
    config = true,
    lazy = false,
    priority = 1001,
  },
}
```

---

### 2.3 Yazi 独立配置
修改 `~/.config/yazi/yazi.toml`，确保在终端独立使用 Yazi 时能正确调用 Neovim。

```toml
[opener]
edit = [
    # [关联] block = true 是核心！让 yazi 挂起并全屏运行 nvim
    { run = 'nvim "$@"', block = true, desc = "Edit", for = "unix" },
]

[open]
prepend_rules = [
    # 针对常见代码/文本文件，强制使用 'edit' 打开器
    { name = "*.txt", use = "edit" },
    { name = "*.md",  use = "edit" },
    { name = "*.lua", use = "edit" },
    { name = "*.py",  use = "edit" },
    { name = "*.sh",  use = "edit" },
    { name = "*.conf", use = "edit" },
    { name = "*.json", use = "edit" },
    # 兜底规则
    { mime = "text/*", use = "edit" },
    { mime = "inode/x-empty", use = "edit" },
]
```

---

### 2.4 环境变量 (Shell)
修改 `~/.zshrc` 或 `~/.bashrc`，设置系统默认编辑器。

```bash
# [关联] 告诉 Yazi 和其他程序默认用 nvim
export EDITOR='nvim'
export VISUAL='nvim'
```

---

### 2.5 终端样式 (Kitty)
修改 `~/.config/kitty/kitty.conf`，实现透明磨砂和光标特效。

```ini
# [关联] 背景不透明度 (配合 Neovim 的 transparent_background)
background_opacity 0.85

# 光标形状：方块
cursor_shape block
# 禁止 Shell 覆盖光标形状
shell_integration no-cursor

# [关联] 光标拖尾特效 (类似 Neovide)
cursor_trail 3
cursor_trail_decay 0.1 0.4
cursor_trail_start_threshold 0
```

---

## 3. 执行命令

完成配置后，请按顺序执行以下命令以应用更改：

1.  **生效环境变量**：
    ```bash
    source ~/.zshrc
    ```
    *(如果是 bash 则 source ~/.bashrc)*

2.  **重载 Kitty 配置**：
    在 Kitty 窗口中按 `Ctrl + Shift + F5`。

3.  **启动 Neovim 安装插件**：
    ```bash
    nvim
    ```
    *LazyVim 会自动拉取插件。若遇报错，输入 `:Lazy` -> 按 `S` 同步。*

4.  **安装语言支持 (LSP)**：
    在 Neovim 中输入：
    ```vim
    :Mason
    ```
    *选择需要的语言（如 python-lsp-server, lua-language-server）并按 `i` 安装。*

---

## 4. 检查成功的方法

请对照下表检查各项功能是否正常：

| 检查项 | 操作 | 预期现象 |
| :--- | :--- | :--- |
| **透明背景** | 打开 nvim，呼出搜索框 (`<Space>/` 或 `<Space>ff`) | 编辑区和搜索弹窗均应透出终端后的壁纸，无灰色背景块。 |
| **光标特效** | 快速移动光标 | 光标应为白色实心方块，且带有平滑的拖尾残影。 |
| **Nvim 调 Yazi** | 在 nvim 中按 `<Space>-` | 屏幕中央弹出 Yazi 悬浮窗。 |
| **Yazi 回传文件** | 在悬浮窗中选中文件按 `Enter` | 悬浮窗关闭，文件在 nvim 当前窗口打开。 |
| **Yazi 调 Nvim** | 在终端输入 `yazi`，选中文件按 `Enter` | nvim 占满全屏打开文件；退出 nvim (`:q`) 后自动回到 Yazi。 |
| **防嵌套** | 在 nvim 内按 `<Ctrl>/` 调出终端，输入 `nvim test.lua` | 文件应在**外部**的主 nvim 窗口打开，而非在小终端窗口内。 |

