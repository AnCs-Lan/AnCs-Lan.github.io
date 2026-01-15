---
title: "LazyVim Plugins"
description: "一些其他的lazyvim插件，可能会时不时加一些新的"
date: 2025-12-27 22:50:35
tags:
   - Arch Linux
categories:
   - Arch Linux
---

在这里放一些配置lazyvim的时候再加装的插件：

1. undotree,用tui树状结构管理文本编辑历史:
```lua
-- 文件路径: ~/.config/nvim/lua/plugins/undotree.lua
return {
  "mbbill/undotree",
  cmd = { "UndotreeToggle", "UndotreeShow", "UndotreeHide", "UndotreeFocus" },
  keys = {
    { "<leader>uu", "<cmd>UndotreeToggle<cr>", desc = "UndoTree Toggle" },
  },
  config = function()
    -- 这里可以添加你对Undotree的自定义配置，例如：
    vim.g.undotree_WindowLayout = 2
    vim.g.undotree_SetFocusWhenToggle = 1
  end,
}
```

2. rendermarkdown,markdown的预览插件：
```lua
-- lua/plugins/markdown.lua
return {
  -- === 插件 1: 渲染美化 ===
  {
    "MeanderingProgrammer/render-markdown.nvim",
    opts = {
      file_types = { "markdown", "Avante" },
      code = {
        sign = false,        -- 去掉左侧细条
        width = "block",     -- 背景铺满整行
        right_pad = 1,
        style = "language",  -- 颜色跟随语法
        border = "thick",    -- 加边框，提升透明背景下的可视度
      },
      heading = {
        sign = false,        -- 去掉 # 号
        icons = { "󰲡 ", "󰲣 ", "󰲥 ", "󰲧 ", "󰲩 ", "󰲫 " },
        width = "block",
        backgrounds = {
          "RenderMarkdownH1Bg", "RenderMarkdownH2Bg", "RenderMarkdownH3Bg",
          "RenderMarkdownH4Bg", "RenderMarkdownH5Bg", "RenderMarkdownH6Bg",
        },
      },
      checkbox = {
        custom = {
          todo = { raw = "[-]", rendered = "󰥔 ", highlight = "RenderMarkdownTodo" },
          important = { raw = "[!]", rendered = " ", highlight = "DiagnosticWarn" },
        },
      },
      anti_conceal = {
        enabled = false, -- 强制隐藏原文，光标移动过去也不显示源码
      },
    },
  },

  -- === 插件 2: 语法检查配置 ===
  {
    "mfussenegger/nvim-lint",
    opts = {
      linters = {
        ["markdownlint-cli2"] = {
          -- 强制指定使用 ~/.config/nvim/markdownlint.yaml
          args = { 
            "--config", 
            vim.fn.expand("~/.config/nvim/markdownlint.yaml"), 
            "--" 
          },
        },
      },
    },
  },
}
```
还有一点点规则放在~/.config/nvim/markdownlint.yaml：
```yaml
# ~/.config/nvim/markdownlint.yaml
default: true

# 禁用列表空格不一致警告
MD030: false
# 禁用单行过长警告
MD013: false
# 允许 HTML 标签
MD033: false
# 不强制第一行必须是标题
MD041: false
```
 全局选项与自动命令 (`lua/config/options.lua` & `autocmds.lua`)
**功能**：解决中文拼写报错红色波浪线，并确保 Conceal 功能生效。

**文件 A: `lua/config/options.lua`**
```lua
-- 让拼写检查把中文(CJK)视为正确，只检查英文拼写
vim.opt.spelllang = "en_us,cjk"
```

**文件 B: `lua/config/autocmds.lua`**
```lua
-- 确保 Markdown 文件开启 Conceal (隐藏源码)
-- 配合 render-markdown 的 anti_conceal = false 使用
vim.api.nvim_create_autocmd("FileType", {
  pattern = { "markdown" },
  callback = function()
    vim.opt_local.conceallevel = 2
  end,
})
```
3. rust的插件安装：
在lazyvim的extras的目录下载lang.rust然后在lazyvim输入:mason，找到并安装rust-analyzer，并且用`pacman`安装`rustup`,
并且用`rustup`安装：`rustup default stable`,从而确保rust的配置文件可以被插件调用阅读



