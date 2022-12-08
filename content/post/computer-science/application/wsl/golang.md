---
draft: false
date: 2022-12-08 08:00:00 +0800
lastmod: 2022-12-08 08:00:00 +0800
title: "使用 WSL 和 Goland 进行 Golang 开发"
summary: "安装和配置 WSL、在 WSL 里安装和配置 Golang、配置 Goland 的过程。"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- wsl
- linux
- golang
- goland
---

### 环境

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> WSL2 Ubuntu-20.04<br/>

### 安装 WSL

Windows 里需要安装 Hyper-V 虚拟机。有没有安装 Hyper-V 可以通过在命令行窗口（CMD、PowerShell）里输入 `systeminfo` 命令查看。

```
...
Hyper-V 要求:     已检测到虚拟机监控程序。将不显示 Hyper-V 所需的功能。
```

完全未安装 WSL 时，执行 `wsl --install` 才会一键安装，这里是手动安装。先以管理员模式运行 PowerShell。

可以通过 `wsl --list --verbose` 命令或者简写的 `wsl -l -v` 命令，查看机器上已经安装的 WSL

```
PS C:\WINDOWS\system32> wsl -l -v
  NAME                   STATE           VERSION
* docker-desktop-data    Stopped         2
  docker-desktop         Stopped         2
```

通过 `wsl --list --online` 命令，查看可用发行版列表。

```
以下是可安装的有效分发的列表。
请使用“wsl --install -d <分发>”安装。

NAME            FRIENDLY NAME
Ubuntu          Ubuntu
Debian          Debian GNU/Linux
kali-linux      Kali Linux Rolling
openSUSE-42     openSUSE Leap 42
SLES-12         SUSE Linux Enterprise Server v12
Ubuntu-16.04    Ubuntu 16.04 LTS
Ubuntu-18.04    Ubuntu 18.04 LTS
Ubuntu-20.04    Ubuntu 20.04 LTS
```

然后通过 `wsl --install -d Ubuntu-20.04` 命令安装 Ubuntu-20.04。

等着下载好，如果进度条不动或者出错了，Ctrl+C 然后重新执行就行。

```
正在下载: Ubuntu 20.04 LTS
安装过程中出现错误。分发名称: 'Ubuntu 20.04 LTS' 错误代码: 0x80072eff
```

```
正在下载: Ubuntu 20.04 LTS
正在安装: Ubuntu 20.04 LTS
已安装 Ubuntu 20.04 LTS。
正在启动 Ubuntu 20.04 LTS…
```

装好后自动会打开一个命令行窗口。

```
Please create a default UNIX user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username:
```

这里需要设置用户名和密码，root 不能用，所以换 kelipute，密码就 123456。

```
Please create a default UNIX user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username: root
adduser: The user `root' already exists.
Enter new UNIX username: kelipute
New password:
Retype new password:
passwd: password updated successfully
Installation successful!
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.10.16.3-microsoft-standard-WSL2 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Dec  8 13:04:08 CST 2022

  System load:  0.0                Processes:             8
  Usage of /:   0.4% of 250.98GB   Users logged in:       0
  Memory usage: 0%                 IPv4 address for eth0: 172.27.168.157
  Swap usage:   0%

0 updates can be installed immediately.
0 of these updates are security updates.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


This message is shown once once a day. To disable it please create the
/home/kelipute/.hushlogin file.
kelipute@keliputeC2021A:~$
```

如果不关闭 Linux 子系统的话，在 Windows 的命令行窗口执行 `wsl -l -v` 命令，就可以看到刚才装的 Ubuntu-20.04，状态是 Running 的。

```
  NAME                   STATE           VERSION
* docker-desktop-data    Stopped         2
  Ubuntu-20.04           Running         2
  docker-desktop         Stopped         2
```

Linux 子系统的命令行窗口，后续可以从 Windows 的开始菜单，搜索 Ubuntu 打开。

### 安装和配置 Golang

在 `$HOME` 目录（`cd ~`） 下载一个 Linux 环境的 go 安装包。

```
sudo wget https://studygolang.com/dl/golang/go1.19.4.linux-amd64.tar.gz
```

把下载下来的压缩包解压到 `/usr/local/` 目录。这里不用自己建一个 go 目录，压缩包里有。

```
sudo tar zxf go1.19.4.linux-amd64.tar.gz -C /usr/local/
```

打开配置文件：`vim ~/.profile`。在最后面追加 Golang 的环境配置。

```
export PATH=$PATH:$GOROOT/bin
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export GOPROXY=https://goproxy.cn,direct
```

保存之后执行 `source ~/.profile` 命令，让配置生效。

这个时候理论上就可以使用 go 命令了，可以使用 `go version` 命令输出 go 的版本，测试一下。

```
go version go1.19.4 linux/amd64
```

### 配置 goland

> 菜单栏 File => Settings... => Build,Execution,Deployment => Run Targets

创建一个 WSL 的，这里创建的是 WSL Ubuntu-20.04。下面的配置，Go Executable 选 `/usr/local/go/bin/go`。GOPATH 选 `/home/kelipute/go`。都是上面在 WSL 里面配置的 go 的环境变量。

然后配置 Goland 右上角的 Run 和 Debug 按钮。偷懒的办法，先本地点一下 Run 或者 Debug。让 Goland 自动创建一个配置，然后进去 **Edit Configurations...** 修改。

选中对应的配置，右边 Run on 默认是 Local machine 这里改成刚才创建的 WSL Ubuntu-20.04，然后把下面的 Build on remote target 勾上。保存之后，这个时候再点 Run 或者 Debug 用的就是 WSL 里面的环境了。

### build 报错

#### run 的时候报错

```
...
dial tcp 172.217.160.113:443: connect: connection refused
...
```

这个是 go mod 下不了包，在 WSL 里面改代理就行。

### debug 报错

#### debug 断点调试的时候报错

```
...
# runtime/cgo
cgo: C compiler "gcc" not found: exec: "gcc": executable file not found in $PATH
...
```

这个提示表示没安装 gcc。使用 `sudo apt install gcc` 命令在 WSL 里面安装 gcc 就行。装完之后可以用 `gcc -v` 命令输出一下 gcc 版本，测试一下。

```
gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/9/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none:hsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 9.3.0-10ubuntu2' --with-bugurl=file:///usr/share/doc/gcc-9/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++,gm2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-9 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none,hsa --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 9.3.0 (Ubuntu 9.3.0-10ubuntu2)
```

### reference（参考）

- [WSL 2 安装](https://wangwangyz.site/archives/1035)
- [Goland WSL2下开发调试](https://blog.csdn.net/qq_38992249/article/details/122393434)
- [goland配合wsl2直接调用wsl2里go环境的方法](https://zhuanlan.zhihu.com/p/378437571)
