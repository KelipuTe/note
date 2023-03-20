---
draft: false
date: 2022-06-12 08:00:00 +0800
lastmod: 2022-06-12 08:00:00 +0800
title: "在 Linux 中配置 Golang 开发环境"
summary: "在 Linux 中配置 Golang 开发环境"
toc: true

categories:
- golang

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- golang
---

### 准备工作

去官网下载压缩包。

- [https://golang.google.cn/dl/](https://golang.google.cn/dl/)
- [https://go.dev/dl/](https://go.dev/dl/)

这里下载的过：

- go1.17.6.linux-amd64.tar.gz
- go1.18.3.linux-amd64.tar.gz

把压缩包解压到 `/usr/local/go` 目录：

```shell
> tar -zxvf go1.17.6.linux-amd64.tar.gz -C /usr/local/go
> tar -zxvf go1.18.3.linux-amd64.tar.gz -C /usr/local/go
```

### 设置系统环境变量

#### 直接执行 export

```shell
> export GOROOT=/usr/local/go
> export PATH=$PATH:$GOROOT/bin
> export PATH=$PATH:$HOME/go/bin
```

这种操作方式只对当前的终端窗口生效，当前窗口关闭后就会恢复原有的 path 配置。

可以注意到有两个 bin 路径:

- 如果是 `make install` 编译安装的可能在 `$GOROOT/bin`，我这里的 protoc 就是这样的
- 如果是 `go install` 安装的 可能在 `$HOME/go/bin`

所以建议是两个 bin 路径都添加一下。

#### 编辑 `/etc/profile` 文件

`sudo vim /etc/profile`，在 `/etc/profile` 文件最后添加：

```shell
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
export PATH=$PATH:$HOME/go/bin
```

保存后，执行 `source /etc/profile` 刷新环境变量。

这种操作方式设置完后即使关闭终端窗口或者重启系统也是生效的。

### 验证

环境变量设置完成，应该就可以用了。

```shell
> go version

go version go1.17.6 linux/amd64
```

### 开启 go mod

```shell
> go env -w GO111MODULE=on
> go env -w GOPROXY=https://goproxy.cn,direct
```

### 参考

- [linux安装go环境](https://blog.csdn.net/qq_44847649/article/details/123048329)
- [ubuntu查看和修改PATH环境变量的方法](https://wenku.baidu.com/view/45b7766dcb50ad02de80d4d8d15abe23482f036c.html)
