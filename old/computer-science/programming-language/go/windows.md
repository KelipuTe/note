## windows环境安装和配置golang开发环境

### 下载

去官方网站（https://golang.org/dl）下一个windows环境的安装包，官方网站打不开就用这个网站（https://golang.google.cn/dl）。这里下载到的安装包是1.17版本的`go1.17.3.windows-amd64.msi`。

### 安装

这里用的是windows11环境，直接运行安装包，选个合适的安装目录，然后一路下一步即可。

### 配置

桌面->我的电脑->鼠标右击->属性->高级系统设置->环境变量->系统变量->path->编辑->添加go安装目录下的bin目录的路径。这里这个目录是`C:\go\bin`。

### 验证

可以在windows的命令行窗口（cmd），使用`go version`命令查看go版本信息。安装步骤成功，则会输出go版本号：`go version go1.17.3 windows/amd64`。

### 环境配置

- 命令`go env`，输出当前go开发包的环境变量状态。
- 命令`go env -w`，设置go开发包的环境变量。

```
> go env
set GO111MODULE=
set GOARCH=amd64
set GOBIN=
set GOCACHE=C:\Users\Administrator\AppData\Local\go-build
set GOENV=C:\Users\Administrator\AppData\Roaming\go\env
set GOEXE=.exe
set GOEXPERIMENT=
set GOFLAGS=
set GOHOSTARCH=amd64
set GOHOSTOS=windows
set GOINSECURE=
set GOMODCACHE=C:\Users\Administrator\go\pkg\mod
set GONOPROXY=
set GONOSUMDB=
set GOOS=windows
set GOPATH=C:\Users\Administrator\go
set GOPRIVATE=
set GOPROXY=https://proxy.golang.org,direct
set GOROOT=c:\go
set GOSUMDB=sum.golang.org
set GOTMPDIR=
set GOTOOLDIR=c:\go\pkg\tool\windows_amd64
set GOVCS=
set GOVERSION=go1.17.3
set GCCGO=gccgo
set AR=ar
set CC=gcc
set CXX=g++
set CGO_ENABLED=1
set GOMOD=NUL
set CGO_CFLAGS=-g -O2
set CGO_CPPFLAGS=
set CGO_CXXFLAGS=-g -O2
set CGO_FFLAGS=-g -O2
set CGO_LDFLAGS=-g -O2
set PKG_CONFIG=pkg-config
set GOGCCFLAGS=-m64 -mthreads -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=C:\Users\ADMINI~1\AppData\Local\Temp\go-build1826627288=/tmp/go-build -gno-record-gcc-switches
```

- GOARCH：处理器架构。
- GOROOT：go开发包的安装目录。
- GOPATH：当前工作目录，在安装的时候会配置默认的。建议不要全局设置，而是随项目设置。
- GOPROXY：代理。
- GOPRIVATE：私有库，不走代理。

### 代理

受网络影响，默认的代理可能连不上，可以换七牛云的代理。使用下面的命令设置代理：

```
> go env -w GO111MODULE=on
> go env -w GOPROXY=https://goproxy.cn,direct
```
