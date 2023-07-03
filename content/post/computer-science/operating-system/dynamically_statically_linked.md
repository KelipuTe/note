---
draft: false
date: 2023-06-10 08:00:00 +0800
title: "动态链接和静态链接"
summary: "动态链接；静态链接；"
toc: true

categories:
  - operating-system(操作系统)

tags:
  - computer-science(计算机科学)
  - operating-system(操作系统)
  - linux
  - c-programming-language
---

## 前言

实践的环境：

- amd64（x86_64）
- windows 11
- vmware workstation pro 16
- ubuntu 22.04
- linux version 5.19.0-41-generic
- gcc version 11.3.0

前置笔记：[运行 ELF 文件](/post/computer-science/operating-system/linux/exec_elf)

## 资料

- [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/program/
- [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/dynamically-statically-linked/

## 正文

### 动态链接

```text
> file hello_world.elf
hello_world.elf: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6d49e18a785eb53d246ca89c5dbce54472ed1584, for GNU/Linux 3.2.0, not stripped
```

之前我们通过 file 命令查看 hello_world.elf 的文件类型时说过，它是 "dynamically linked" 的，也就是动态链接的。
如果一个 elf 可执行文件是动态链接的。那么，它必定有一个解释器，这个解释器就是 linux 动态链接器。
在这里就是 "interpreter /lib64/ld-linux-x86-64.so.2" 这一段。interpreter 就是解释器的意思，后面的是解释器的路径。

有的时候动态链接的信息可以从 file 命令的结果里可以直接看见。
如果没有的话，也可以通过 readelf 命令，从 elf 文件的 program headers 中找到。
这里截取了 readelf 命令的输出中关于解释器的那一部分。

```
Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```

前面我们也提到过 ".interp" 段（上面的 INTERP）里面，存的就是程序解释器（elf 解释器）的路径。

这里的就是这个 "interpreter: /lib64/ld-linux-x86-64.so.2"，它就是 linux 动态链接器。
它的功能是，把 elf 程序所依赖的动态库（共享库）加载并初始化。

如果 elf 可执行程序的类型是动态链接的话，那么这个程序加载到内存运行时，操作系统会把控制权先交给动态链接器。
动态链接器加载完该程序所依赖的动态库并初始化相应的必要操作后，才会把控制权交给程序的入口地址。
如果这个程序没有 ".interp" 段，就会直接从程序的入口地址直接执行。

也就是说，linux 在使用 execve() 加载程序时。不管 elf 文件的具体类型是什么。
只要检测出含有 ".interp" 段，那么，linux 就会先运行动态链接器。
动态链接器运行完之后，才会将控制权交给 elf 程序的入口地址。

可以通过 ldd 命令查看文件依赖的动态库。
关于 ldd 命令具体怎么用，可以看 "ldd(1) - print shared object dependencies"。

```
> ldd hello_world.elf
	linux-vdso.so.1 (0x00007fffafdd6000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f6d35e00000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f6d3620c000)
```

"/lib64/ld-linux-x86-64.so.2" 就是动态链接器。
通过 `file /lib64/ld-linux-x86-64.so.2` 命令，可以查看它的文件类型。

结果是："/lib64/ld-linux-x86-64.so.2: symbolic link to /lib/x86_64-linux-gnu/ld-linux-x86-64.so.2"
其中 "symbolic link to" 表示它是个软链接。链接到的 "ld-linux-x86-64.so.2" 是个特殊的共享库文件，因为它可以执行。

"/lib/x86_64-linux-gnu/libc.so.6" 就是依赖的动态库。它是 c 的运行库，封装了很多操作系统的接口。
libc.so.6 是 GNU c 库（glibc）的一个重要组成部分，是大多数 linux 发行版使用的标准 c 库。

linux 系统中，c 库的命名规则通常遵循 libc.so.<版本> 的模式，libc.so.6 就表示它是 c 库的第 6 版。

动态库允许多个程序在内存中共享同一实例，减少内存消耗，并使动态库的更新变得容易，而不需要重新编译所有依赖它的程序。
这样程序就可以以模块的形式独立开发，并且方便后续的维护和升级。

动态库的查找路径："/usr/lib"、"/usr/lib64" 存储的是操作系统的共享库；
"/usr/local/lib"、"/usr/local/lib64"存储的是第三方的共享库。

"linux-vdso.so.1"，它是比较特殊的一个共享库，它不在文件系统中，而是在内核中。
详细的可以看："vdso(7) - overview of the virtual ELF dynamic shared object"。

### 编写和使用动态库

我在 "{demo-c}/demo-in-linux/dynamically-statically-linked/" 目录下准备了样例代码。

dynamically_library.c 是动态库本体。里面准备了一个 print_hello_world()。
编码完成后使用 `gcc -fPIC -shared dynamically_library.c -o dynamically_library.so` 命令，编译成动态库。

动态库一般有两种使用方式：编译时链接动态库（写代码的时候引入头文件，头文件里面是声明和定义）；程序中显示调用动态库。

首先是编译时使用动态库，我们在 use_when_gcc.c 里面直接声明然后调用 print_hello_world()。
这个时候直接使用 gcc 编译肯定是不行的，因为标准库里面没有 print_hello_world()。

需要使用 `gcc use_when_gcc.c ./dynamically_library.so -o use_when_gcc.elf` 命令，在编译时连接动态库。

在程序中显示调用动态库比较麻烦，需要借助几个系统调用：dlopen()、dlsym()。代码在 use_in_code.c 里面。
这个编译的时候这样写 `gcc use_in_code.c -ldl -o use_in_code.elf`，不需要连接动态库。

### 静态链接

如果通过 file 命令查看静态链接的 elf 可执行文件的话，结果中会有 "statically linked"。
静态链接的程序，会直接从程序的入口地址开始执行。

我这里准备了 2 个文件，statically_library.c、statically_use.c

```c
// statically_library.c
int returnVal (int v){
  return v;
}
```

```c
// statically_use.c
#include <stdio.h>

int main() {
  returnVal(1);
  return 0;
}
```

使用 gcc 先将两个文件编译成可重定位文件。
`gcc -c statically_library.c -o statically_library.o`。
`gcc -c statically_use.c -o statically_use.o`。

编译 statically_use.c 的时候会报错，这个忽略就好，这玩意肯定是有问题的。

```text
statically_use.c: In function ‘main’:
statically_use.c:4:3: warning: implicit declaration of function ‘returnVal’ [-Wimplicit-function-declaration]
    4 |   returnVal(1);
      |   ^~~~~~~~~
```

这个时候我们使用 objdump 命令分别看一下 statically_library.o 和 statically_use.o。
这里截取了 ".text" 段的内容，要和下面 statically_call.elf 的对比着看的。

```text
> objdump -s statically_library.o

Contents of section .text:
 0000 f30f1efa 554889e5 897dfc8b 45fc5dc3  ....UH...}..E.].
```

```text
> objdump -s statically_use.o

Contents of section .text:
 0000 f30f1efa 554889e5 bf010000 00b80000  ....UH..........
 0010 0000e800 000000b8 00000000 5dc3      ............].  
```

然后，这里用的不是 gcc 了，用的是静态链接器 ld，用 gcc 的时候，它会自动调用 ld。
链接 statically_library.o 和 statically_use.o 并设置程序入口为 main。
`ld statically_library.o statically_use.o -e main -o statically_call.elf`。

这个时候就得到了一个 elf 可执行文件。这玩意跑不起来，因为除了这两段代码别的啥都没有。
用 file 命令看一下，可以看到输出的结果里面有 "statically linked"。

```text
> file statically_call.elf 
statically_call.elf: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped
```

再使用 objdump 命令看一下 statically_call.elf。可以发现 ".text" 段的内容，就是上面两个文件拼起来的。

```text
> objdump -s statically_call.elf

Contents of section .text:
 401000 f30f1efa 554889e5 897dfc8b 45fc5dc3  ....UH...}..E.].
 401010 f30f1efa 554889e5 bf010000 00b80000  ....UH..........
 401020 0000e8d9 ffffffb8 00000000 5dc3      ............].  
```

静态链接器会将输入的目标文件进行合并，把 ".text" 段、".data" 段等进行合并，合并后会重新计算段的大小、偏移位置。
同时静态链接器还要完成符号解析（确保所有的符号被正确地连接）、重新定位（调整内存地址和引用）等操作。
