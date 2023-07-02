---
draft: false
date: 2023-06-10 08:00:00 +0800
title: "程序"
summary: "什么是程序；从源代码到可执行文件；编译过程产生的几个文件；程序的运行过程是怎样的；"
toc: true

categories:
  - operating-system(操作系统)

tags:
  - computer-science(计算机科学)
  - operating-system(操作系统)
---

## 前言

实践的环境：

- amd64（x86_64）
- windows 11
- vmware workstation pro 16
- ubuntu 22.04
- linux version 5.19.0-41-generic
- gcc version 11.3.0

## 资料

- [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/program/

## 正文

### 什么是程序

程序是计算机可以执行的一条或者一组指令，不管怎样，程序最终要在计算机物理硬件上跑起来的。

程序有很多种形态：计算机可以直接执行，但是，不方便开发人员阅读和处理的二进制机器码是程序；
计算机不能直接执行，但是，方便开发人员阅读和处理的由各种编程语言编写的源代码也是程序。

程序还有一种特别的形态，即由各种编程语言编写的源代码编译出来的可执行文件。
比如：windows 系统中 exe 可执行文件；linux 系统中的 elf 可执行文件。

那么问题来了，程序有这么多种形态，它们之间是怎么能正确的相互转化的？
答案是，通过抽象，而且是好多层的抽象。因为分类方式不同，会有不同的分层，所以下面说的，都是我的理解。

首先是，最上面的应用软件层和最下面的物理硬件层。
这两层比较好理解，应用软件层就是我们使用的各式各样的应用程序，物理硬件层就是这些应用程序最终运行时所依托的计算机物理硬件。

那么，中间有几层呢？这里不一定，我目前把中间分成 3 层。
操作系统肯定有一层，这个也好理解，我们使用计算机的时候都需要先安装一个操作系统。然后，在操作系统里面安装我们需要的应用程序。

一般来说，开发人员开发的应用程序是不能直接操作操作系统的，使用的是操作系统提供的系统接口。
开发人员通过操作系统提供的系统接口实现应用程序的功能。所以，这里有一层。

同理，开发人员开发的操作系统也是不能直接操作物理硬件的，使用的是物理硬件提供的固件接口。
固件是指嵌入硬件设备中的软件，开发人员通过固件接口开发操作系统。所以，这里也有一层。

### 从源代码到可执行文件

对于应用软件的开发人员来说，一般都是基于操作系统进行开发的。所以，我这里的讨论，到操作系统就为止了。
不是说下层就不重要了，而是由于抽象层的存在，我们可以规避掉下层茫茫多的细节，专心讨论上层的问题。

上面提到过，开发人员开发的应用程序，使用的是操作系统提供的系统接口。系统接口也叫系统调用（system call）。
这个玩意，现在就解释有点太早了，在 [运行 ELF 文件]() 这篇里面会详细的说。

#### 实践的环境

这里，我借助 linux 系统和 c 语言来讨论这个问题。我这里实践的环境如下。
cpu 是 amd64（x86_64）架构，操作系统是 windows 11。这个可以通过 `systeminfo` 命令查看。

然后，在 windows 中，使用 vmware workstation pro 16，搭建 ubuntu 22.04 虚拟机。
ubuntu 22.04 的 linux 版本是 linux version 5.19.0-41-generic。这个可以通过 `cat /proc/version` 命令查看。

然后，在 linux 中安装 gcc 编译器，版本是 gcc version 11.3.0 (ubuntu 11.3.0-1ubuntu1~22.04)。

#### [在 Linux 中使用 C 语言进行编程的注意点](/post/computer-science/operating-system/linux/notice)

#### 编译

可执行文件是操作系统提供的对于程序的抽象，内容包括程序指令和程序数据。
程序指令就是前面说过的指令集里面的指令，程序数据就是二进制的数据。
按照操作系统提供的对于程序的抽象编写出来的程序，操作系统才看得懂。

可执行文件就相当于一个操作手册，操作手册上记录了每一步要干什么。
手册上是用文字描述要干什么的，实际操作的时候，是一个个具体动作，指令集记录的就是文字描述和具体动作的对应关系。
操作系统拿着操作手册，再对照指令集，就可以知道需要在硬件系统上做什么具体动作。

想从源代码变成可执行文件，通常要进行编译。
在 linux 系统中就是，c 源码文件通过 gcc 编译器的编译得到 elf 可执行文件。
在 windows 系统中就是，c 源码文件通过 gcc 编译器的编译得到 exe 可执行文件。这里讨论的是在 linux 系统中的情况。

通常使用 gcc 命令 `gcc xxx.c -o xxx` 一步就完成了。但是，这一步里面其实有四个步骤：预处理、编译、汇编、链接。
关于 gcc 命令具体怎么用，可以看 "gcc(1) - GNU project C and C++ compiler"。

这里会用到 "-E"、"-S"、"-c"、"-o" 这几个参数。
"-E" 表示会进行预处理；"-S" 表示会进行预处理、编译，得到汇编代码；
"-c" 表示会进行预处理、编译、汇编，得到可重定位文件；"-o" 表示指定目标名称；
如果什么参数都不写，那就会预处理、编译、汇编、链接一步到位，得到可执行文件。

- 预处理（preprocessing）：`gcc -E xxx.c -o xxx.i`，预处理器会在源码的基础上增加一些代码。
- 编译（compilation）：`gcc -S xxx.i -o xxx.s`，编译器通过编译代码（词法分析、语法分析等）得到汇编代码。
- 汇编（assemble）：`gcc -c xxx.s -o xxx.o`，汇编器把汇编代码转化为机器指令，这一步得到的是可重定位文件。
- 链接（link）：`gcc xxx.o -o xxx.elf`，连接器把各个模块链接起来，组织成为可执行文件。

在链接中，会把函数的名字和变量的名字都称为符号（symbol）。源代码中的函数的名字和变量的名字，无论长度多少，最终都会转换为符号。

我在 {demo-c}/demo-in-linux/program/ 目录准备了五个文件。
hello_world.c、hello_world.i、hello_world.s、hello_world.o、hello_world.elf。
分别对应上面的源码文件，还有每个命令执行之后生成的文件。

### 编译过程产生的几个文件

首先，我们使用 file 命令，依次看一下这 5 个文件的文件类型。
关于 file 命令具体怎么用，可以看 "file(1) - determine file type"。

```text
> file hello_world.c
hello_world.c: C source, ASCII text, with CRLF line terminators
> file hello_world.i
hello_world.i: C source, ASCII text
> file hello_world.s
hello_world.s: assembler source, ASCII text
> file hello_world.o
hello_world.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
> file hello_world.elf
hello_world.elf: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6d49e18a785eb53d246ca89c5dbce54472ed1584, for GNU/Linux 3.2.0, not stripped
```

这里解释一下。
hello_world.c 和 hello_world.i 都是 c 源码文件（C source）；
hello_world.s 是汇编源码文件（assembler source）；
hello_world.o 是可重定位文件（relocatable）；
hello_world.elf 是可执行文件（executable）。

特别注意一下 hello_world.elf，它的内容有点多的。
"dynamically linked" 表示它是动态链接的。
"interpreter /lib64/ld-linux-x86-64.so.2" 表示它有一个解释器，解释器的路径是 "/lib64/ld-linux-x86-64.so.2"。
这两个玩意，现在就解释有点太早了，在 [运行 ELF 文件]() 这篇里面会详细的说。

这里，再用 objdump 命令观察一下 hello_world.o 文件的具体内容。
关于 objdump 命令具体怎么用。可以看 "objdump(1) - display information from object files"

这里会用到 "-h"、"-s"、"-d" 这几个参数。
"-h" 表示输出段表；"-s" 表示输出段表中每个段的详细内容；"-d" 表示输出 ".text" 段对应的汇编代码。
命令的输出我放在 {demo-c}/demo-in-linux/program/ 目录的 hello_world_objdump.md 文件内。

分别解释一下。

命令输出的结果的第一行都是 `hello_world.o:     file format elf64-x86-64`，这个是文件的类型。

"-h" 输出的段表。

- ".text" 是代码段，它用于存储程序指令（程序的代码）；
- ".data"、".bss"、".rodata" 都是数据段，它们用于存储程序数据；
- ".bss" 是未初始化数据段；".rodata" 是只读数据段；

"-s" 输出的段表中每个段的详细内容。这里截取了 ".text" 段。

```text
Contents of section .text:
 0000 f30f1efa 554889e5 488d0500 00000048  ....UH..H......H
 0010 89c7e800 000000b8 00000000 5dc3      ............].  
```

- 左边第 1 列是虚拟地址，表示内存偏移量。
- 中间的 4 列是指令码，表示程序指令的内容（16 进制机器指令）。
- 最右边 1 列是 ASCII 码，是程序指令的 ASCII 文本。

"-d" 输出的 ".text" 段对应的汇编代码。

```text
Disassembly of section .text:

0000000000000000 <main>:
   0:	f3 0f 1e fa          	endbr64 
   4:	55                   	push   %rbp
   5:	48 89 e5             	mov    %rsp,%rbp
   8:	48 8d 05 00 00 00 00 	lea    0x0(%rip),%rax        # f <main+0xf>
   f:	48 89 c7             	mov    %rax,%rdi
  12:	e8 00 00 00 00       	call   17 <main+0x17>
  17:	b8 00 00 00 00       	mov    $0x0,%eax
  1c:	5d                   	pop    %rbp
  1d:	c3                   	ret    
```

"Disassembly of section .text"，表示程序指令的内容，以及对应的汇编代码。这里和 "-s" 输出的结果对应。
上面 "-s" 输出的结果里的 "Contents of section .text" 里，第一行第二段的 "f30f1efa"，就对应这里的 "0:    f3 0f 1e fa"。
下面一行的 "4:    55" 和后面的 "push %rbp"，就表示 "55"（0x55）反汇编后对应的汇编指令就是 "push %rbp"。

然后，再用 objdump 命令观察一下 hello_world.elf 文件。命令的输出也放在 hello_world_objdump.md 文件里面。
这里可以看到，hello_world.elf 的内容是比 hello_world.o 要多出不少的。

而且，通过 hello_world.elf 的内容，我们可以发现。

第一点：hello_world.elf 的 ".text" 段的第一个函数不是 main 函数，而是 "_start" 函数。
"_start" 函数会调用 "__libc_start_main" 函数进行一些必要的初始化操作，然后再调用 main 函数。
也就是说，main 函数并不是程序的入口。

第二点：一个简单的 hello world 程序没有看上去的那么简单。

### 可执行文件

关于 elf 可执行文件的详细的内容。我放到这篇里面去了。
[ELF 文件](/post/computer-science/operating-system/linux/elf)

### 程序的运行过程是怎样的

关于 elf 可执行文件的运行过程的详细的内容。我放到这篇里面去了。
[运行 ELF 文件](/post/computer-science/operating-system/linux/exec_elf)
