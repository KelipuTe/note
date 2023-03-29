## windows环境配置c语言开发环境

### 下载安装包

访问MinGW的主页（https://www.mingw-w64.org）。在左侧导航标签找到Downloads。点击Downloads，进入下载页，可以看见很多开发包。

以前用的是MingW-W64-builds（https://sourceforge.net/projects/mingw-w64/），但是新版的这玩意是在用不明白。

所以这里推荐Cygwin（https://cygwin.com/install.html），进入下载页后，下载64位windows版本的安装器（setup-x86_64.exe）。

### 安装cygwin

- 这里是windows11环境。双击运行setup-x86_64.exe，下一步。
- Choose A Download Source界面，选Install from Internet，下一步。
- Select Root Install Directory界面，设置安装目录，选All Users，下一步。
- Select Root Package Directory界面，设置文件下载目录，下一步。
- Select Your Internet Connection界面，选Use System Proxy Settings，下一步。然后，会有一个界面让你选下载源镜像，本人这里选的华为的（https://mirrors.huaweicloud.com/cygwin/）。
- Progress界面，等待下载完成，下一步。
- Select Packages界面，在Search输入框里，分别搜索gcc-core、gcc-g++、make、gdb、binutils。然后在下拉列表选择版本，本人选的各自列表的倒数第1个非测试版本。下一步。
- 然后等待安装结束就行。
- 安装结束后，桌面->我的电脑->鼠标右击->属性->高级系统设置->环境变量->系统变量->path->编辑->添加cygwin安装目录下bin目录的路径。这里这个目录是`C:\cygwin64\bin`。

### 验证

打开windows命令行窗口（cmd），分别检查gcc和gdb命令是否可用。输出一下版本信息即可。

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