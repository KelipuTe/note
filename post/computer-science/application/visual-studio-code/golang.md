---
draft: false
date: 2021-11-17 08:00:00 +0800
lastmod: 2023-04-11 08:00:00 +0800
title: "在 Visual Studio Code 中，配置 Golang 开发环境"
summary: "在 Visual Studio Code 中，配置 Golang 开发环境"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- application(应用)
- golang
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- go1.19 windows/amd64
- Visual Studio Code 1.77.1

公共的操作放在这篇里面：[在 Windows 11 中，安装和配置 Visual Studio Code](/post/computer-science/application/visual-studio-code/windows)

## 正文

建议 VSCode 按工作区去配置，这样不同项目互不影响。

### 安装 Golang

先把 go 开发环境装好：[在 Windows 11 中，配置 Golang 开发环境](/post/computer-science/programming-language/golang/windows)

### 常用设置

- Go: Format Tool。格式化工具。

### 扩展

- （可选）安装 Chinese (Simplified) (简体中文) Language Pack for Visual Studio Code 组件（简体中文语言包）。
- 安装 Go 组件。
- 安装 Code Runner 组件。
- （可选）安装 Git Graph 组件（查看 Git 树，执行 Git 操作）。
- （可选）安装 Git History 组件（查看 Git 历史）。
- （可选）安装 GitLens 组件（查看代码作者）。
- （可选）安装 Markdown All in One 组件。
- 其他的工具是 go 提供的，会在右下角弹出提示框，点击安装即可。

### 工作区设置

有些设置项可以根据语言进行不同的配置。那种的就不是点 "在 settings.json 中编辑" 而是 "编辑 {语言} 的设置"。

比如，Editor: Code Actions On Save 这个设置。点击 "在 settings.json 中编辑" 不会出现下面的 `"[go]": {}` 那样的参数。点击 "编辑 go 的设置" 的时候才会出现。

```
{
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  }
}
```

```
  "[go]": {
    "editor.codeActionsOnSave": {
      "source.organizeImports": true
    }
  }
```

### 断点调试

找一个源码文件，打上断点。

在最上面的菜单栏，点击 "启动调试"。第一次点的时候，会在右下角弹出提示框，提示安装 go dlv。

安装好之后，点击 "启动调试"，理论上就可以了。

### 代码格式化

设置 Go: Format Tool 为 goformat。其他的理论上不用改，go 提供的格式化工具自己有一套代码格式标准。

### 使用 Code Runner 运行代码

安装 Code Runner 组件后，右上角的 "运行按钮" 那里，会多一个 Run Code 的选项。go 这里应该是本来没有这个按钮，现在会多一个按钮出来。

选中 hello_world.go 文件，然后，点击右上角的运行按钮，选择 Run Code 选项。这个操作，会在下面的 "终端" 子窗口里面，输出以下内容，相当于编译并运行了 hello_world.go 文件。

```
[Running] go run "d:\workspace\demo-golang\demo\helloworld\main.go"
hello, world
```
