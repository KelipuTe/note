---
draft: false
date: 2022-05-09 08:00:00 +0800
lastmod: 2023-04-10 08:00:00 +0800
title: "在 Windows 11 中，配置 C 语言开发环境"
summary: "在 Windows 11 中，配置 C 语言开发环境"
toc: true

categories:
- ccpp

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- ccpp
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- x86_64-pc-cygwin
- gcc 版本 11.2.0 (GCC)

## 正文

### 下载安装包

访问 [MinGW 的主页](https://www.mingw-w64.org)。在页面左侧的导航中，找到 Downloads。点击 Downloads，进入下载页，可以看见很多开发包。

我以前用的是 MingW-W64-builds，一般在 SourceForge 上下载（[下载页面](https://sourceforge.net/projects/mingw-w64/)）。但是，新版的玩不明白，就放弃了。

所以，这里推荐 Cygwin。下载页有 Cygwin 主页的 [链接](https://cygwin.com)。在页面左侧的导航中，找到 Cygwin 下面的 Install Cygwin。点击 Install Cygwin，进入下载页，下载 64 位 windows 版本的安装包（setup-x86_64.exe）。

### 安装 cygwin

我这里是 Windows 11 家庭版。建议使用管理员权限运行安装包 setup-x86_64.exe。

- 运行 setup-x86_64.exe，下一步。
- Choose A Download Source 界面，选 Install from Internet，下一步。
- Select Root Install Directory 界面，设置安装目录，选 All Users，下一步。
- Select Root Package Directory 界面，设置文件下载目录，下一步。
- Select Your Internet Connection 界面，选 Use System Proxy Settings，下一步。然后，会有一个界面让你选下载源镜像，我这里选的华为的（"https://mirrors.huaweicloud.com/cygwin/"）。
- Progress 界面，等待下载完成，下一步。
- Select Packages 界面，在 Search 输入框里，分别搜索 gcc-core、gcc-g++、make、gdb、binutils。然后，在下拉列表选择版本。本人选的各自列表的倒数第 1 个非测试版本。下一步。
- 然后，等待安装结束。

安装结束后，需要配置环境变量。

- 在 Windows 桌面，鼠标右击 "我的电脑"，在弹出的菜单中，点击 "属性"，打开 "设置" 窗口。
- 在设置窗口中，找到并点击 "高级系统设置"，打开 "系统属性" 窗口。
- 在系统属性窗口中，切换到 "高级" 标签页，然后点击标签页最下面的 "环境变量"，打开 "环境变量" 窗口。
- 在环境变量窗口下面的 "系统变量" 里找到 path，然后点击 "编辑，打开 "编辑环境变量" 窗口"。
- 添加 "cygwin 的安装目录下的 bin 目录的路径"。我这里，这个目录是 "C:\cygwin64\bin"。

配置完环境变量之后，可能需要重启一下 Windows。

### 验证能不能用

打开命令行窗口，CMD 或者 PowerShell 都可以。分别检查 gcc 和 gdb 命令是否可用。输出一下版本信息即可。

```
> gcc -v

Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-pc-cygwin/10/lto-wrapper.exe
Target: x86_64-pc-cygwin
Configured with: /mnt/share/cygpkgs/gcc/gcc.x86_64/src/gcc-10.2.0/configure --srcdir=/mnt/share/cygpkgs/gcc/gcc.x86_64/src/gcc-10.2.0 --prefix=/usr --exec-prefix=/usr --localstatedir=/var --sysconfdir=/etc --docdir=/usr/share/doc/gcc --htmldir=/usr/share/doc/gcc/html -C --build=x86_64-pc-cygwin --host=x86_64-pc-cygwin --target=x86_64-pc-cygwin --without-libiconv-prefix --without-libintl-prefix --libexecdir=/usr/lib --with-gcc-major-version-only --enable-shared --enable-shared-libgcc --enable-static --enable-version-specific-runtime-libs --enable-bootstrap --enable-__cxa_atexit --with-dwarf2 --with-tune=generic --enable-languages=c,c++,fortran,lto,objc,obj-c++ --enable-graphite --enable-threads=posix --enable-libatomic --enable-libgomp --enable-libquadmath --enable-libquadmath-support --disable-libssp --enable-libada --disable-symvers --with-gnu-ld --with-gnu-as --with-cloog-include=/usr/include/cloog-isl --without-libiconv-prefix --without-libintl-prefix --with-system-zlib --enable-linker-build-id --with-default-libstdcxx-abi=gcc4-compatible --enable-libstdcxx-filesystem-ts
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 10.2.0 (GCC)
```

```
> gdb -v

GNU gdb (GDB) (Cygwin 10.2-1) 10.2
Copyright (C) 2021 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```
