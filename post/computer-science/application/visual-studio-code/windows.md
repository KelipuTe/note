---
draft: false
date: 2022-05-11 08:00:00 +0800
lastmod: 2023-04-10 08:00:00 +0800
title: "在 Windows 11 中，安装和配置 Visual Studio Code"
summary: "在 Windows 11 中，安装和配置 Visual Studio Code"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- application(应用)
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- Visual Studio Code 1.77.1

## 正文

Visual Studio Code 在笔记中，会简写为 VSCode。

### 安装 VSCode

去 VSCode [官网](https://code.visualstudio.com/) 下个安装包，然后装一下就好了。可能需要使用管理员权限运行安装包。

### 工作区配置

建议 VSCode 按工作区去配置，这样不同项目互不影响。

打开 "设置" 标签页。下面两种方式都可以。

- 在最上面的菜单栏，点击 "文件"，在出现的下拉菜单中，点击 "首选项"。然后，在出现的菜单中，点击 "设置"。
- 使用快捷键 "Ctrl+,"。

在设置标签页最上面，默认选中的是用户标签页。往下翻，随便找一个 "在 settings.json 中编辑" 点一下。

这个时候会打开一个 settings.json 文件，我的这个文件的位置在 C:\Users\kelipute\AppData\Roaming\Code\User\settings.json。它是当前 Windows 用户的配置，也就是说它是全局配置文件。

如果想给当前工作区配置的话。在设置标签页最上面，先切换到工作区标签页。往下翻，随便找一个 "在 settings.json 中编辑" 点一下。

这个操作会自动在项目根目录的 ".vscode/" 目录（没有会自动创建）下，创建一个 settings.json 文件。这个 settings.json 文件，就是这个工作区自己的设置文件了。

### 常用设置

在设置标签页上面的输入框中可以输入设置项的名字进行搜索。

- Editor: Font Size。控制字体大小(像素)。
- Debug › Console: Font Size。控制调试控制台中的字号(以像素为单位)。
- Terminal › Integrated: Font Size。控制终端的字号(以像素为单位)。

- Editor: Word Wrap。控制折行的方式。

- Editor: Detect Indentation。控制在基于文件内容打开文件时是否自动检测 Editor: Tab Size 和 Editor: Insert Spaces。
- Editor: Insert Spaces。按 Tab 时插入空格。当 Editor: Detect Indentation 打开时，将根据文件内容替代此设置。
- Editor: Tab Size。一个制表符等于的空格数。当 Editor: Detect Indentation 打开时，将根据文件内容替代此设置。
- Editor: Render Whitespace。控制编辑器在空白字符上显示符号的方式。

- Editor: Format On Save。在保存时格式化文件。格式化程序必须可用，延迟后文件不能保存，并且编辑器不能关闭。
- Editor: Code Actions On Save。要在保存时运行的代码操作种类。点了 "在 settings.json 中编辑" 会跳到 settings.json 文件并弹出可供选择的设置项列表，设置项都有注释可以看。

- Editor › Suggest: Snippets Prevent Quick Suggestions。控制活动代码段是否阻止快速建议。

把设置项从默认值改成自定义值之后，就会在 settings.json 文件里面添加一条对应的参数。

有些设置项可以根据语言进行不同的配置。那种的就不是点 "在 settings.json 中编辑" 而是 "编辑 {语言} 的设置"。

### 安装 VSCode 扩展

打开左侧的 "扩展" 工具栏。下面三种方式都可以。

- 在最上面的菜单栏，点击 "文件"，在弹出的菜单中，点击 "首选项"，在弹出的菜单中，点击 "扩展"。
- 在最左边的一列按钮里，找到并点击 "扩展"。
- 使用快捷键 "Ctrl+Shift+X"。

搜索并安装扩展。可以直接在列表里找组件，也可以在扩展工具栏最上面的输入框输入组件的名字进行搜索。

### 调出命令面板

下面两种方式都可以。

- 在最上面的菜单栏，点击 "查看"，在弹出的菜单中，点击 "命令面板"
- 使用快捷键 "Ctrl+Shift+P"。

### 配置 Code Runner 组件

历史笔记，最新的配置过程，这个步骤可以跳过。

在最上面的菜单栏，点击 "文件"，在出现的下拉菜单中，点击 "首选项"。然后，在出现的菜单中，点击 "扩展"，打开 "设置" 标签页。在设置标签页最上面的输入框中，输入 "@ext:formulahendry.code-runner"。

也可以先打开扩展工具栏。在扩展工具栏中，选择 Code Runner。在弹出的 C/C++ 组件的标签页，点击的标题下面的设置图标，在弹出的下拉菜单中，选择扩展设置，打开设置标签页。

这两个方式都可以找到 Code Runner 的设置。找到之后，勾选 Code-runner: Ignore Selection 和 Code-runner: Run In Terminal 这两项。
