---
draft: false
date: 2021-11-17 08:00:00 +0800
lastmod: 2023-04-11 08:00:00 +0800
title: "在 Windows 11 中，配置 Golang 开发环境"
summary: "在 Windows 11 中，配置 Golang 开发环境"
toc: true

categories:
- golang

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- golang
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- go1.19 windows/amd64

## 正文

### 下载

去 Golang 的 [官方网站](https://golang.org) 下一个 Windows 环境的安装包。官方网站打不开就用 [这个网站](https://golang.google.cn/)。我这里下载到的安装包是 1.19 版本的 go1.19.3.windows-amd64.msi。

### 安装

我这里是 Windows 11 家庭版。建议使用管理员权限运行安装包。选个合适的安装目录，然后一路下一步即可。

安装结束后，需要配置环境变量。

- 在 Windows 桌面，鼠标右击 "我的电脑"，在弹出的菜单中，点击 "属性"，打开 "设置" 窗口。
- 在设置窗口中，找到并点击 "高级系统设置"，打开 "系统属性" 窗口。
- 在系统属性窗口中，切换到 "高级" 标签页，然后点击标签页最下面的 "环境变量"，打开 "环境变量" 窗口。
- 在环境变量窗口下面的 "系统变量" 里找到 path，然后点击 "编辑，打开 "编辑环境变量" 窗口"。
- 添加 "go 的安装目录下的 bin 目录的路径"。我这里，这个目录是 "C:\go\bin"。

配置完环境变量之后，可能需要重启一下 Windows。

### 验证能不能用

打开命令行窗口，CMD 或者 PowerShell 都可以。检查 go 命令是否可用。输出一下版本信息即可。

```
> go version

go version go1.19 windows/amd64
```

### 环境配置

- `go env` 命令，输出当前 go 开发包的环境变量。
- `go env -w` 命令，设置当前 go 开发包的环境变量。

```
> go env

set GO111MODULE=on
set GOARCH=amd64
set GOBIN=
set GOCACHE=C:\Users\kelipute\AppData\Local\go-build
set GOENV=C:\Users\kelipute\AppData\Roaming\go\env
set GOEXE=.exe
set GOEXPERIMENT=
set GOFLAGS=
set GOHOSTARCH=amd64
set GOHOSTOS=windows
set GOINSECURE=
set GOMODCACHE=C:\Users\kelipute\go\pkg\mod
set GONOPROXY=
set GONOSUMDB=
set GOOS=windows
set GOPATH=C:\Users\kelipute/go
set GOPRIVATE=
set GOPROXY=https://goproxy.cn,direct
set GOROOT=C:\go1.19
set GOSUMDB=sum.golang.org
set GOTMPDIR=
set GOTOOLDIR=C:\go1.19\pkg\tool\windows_amd64
set GOVCS=
set GOVERSION=go1.19
set GCCGO=gccgo
set GOAMD64=v1
set AR=ar
set CC=gcc
set CXX=g++
set CGO_ENABLED=1
set GOMOD=NUL
set GOWORK=
set CGO_CFLAGS=-g -O2
set CGO_CPPFLAGS=
set CGO_CXXFLAGS=-g -O2
set CGO_FFLAGS=-g -O2
set CGO_LDFLAGS=-g -O2
set PKG_CONFIG=pkg-config
set GOGCCFLAGS=-m64 -mthreads -Wl,--no-gc-sections -fmessage-length=0 -fdebug-prefix-map=C:\Users\kelipute\AppData\Local\Temp\go-build3774566253=/tmp/go-build -gno-record-gcc-switches
```

- GOARCH：处理器架构。
- GOROOT：go 开发包的安装目录。
- GOPATH：当前工作目录，在安装的时候会配置默认的。建议不要全局设置，而是随项目设置。
- GOPROXY：代理。
- GOPRIVATE：私有库，不走代理。

### 设置代理

受网络影响，默认的代理可能连不上。使用命令 `go env -w GOPROXY=https://goproxy.cn,direct` 设置为七牛云的代理。

### go modules

使用命令 `go env -w GO111MODULE=on` 开启 go modules。go modules 是用于依赖管理的，以前的项目用的是 go path。现在，新的项目基本不用 go path 了，go modules 更方便。
