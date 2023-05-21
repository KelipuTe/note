---
draft: false
create_date: 2021-12-04 08:00:00 +0800
date: 2023-02-06 08:00:00 +0800
title: "什么是程序"
summary: "什么是程序；编译的过程；可执行文件；"
toc: true

categories:
- operating-system(操作系统)

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- linux-c
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- VMware Workstation Pro 16
- Ubuntu 22.04
- Linux 5.19.0-32-generic x86_64
- gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0

## 资料

- [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/helloworld/

## 正文

### 程序

存储在磁盘上的静态代码文件或者由静态代码文件编译出来的可执行文件都可以说是程序。

### 编译

通常说的编译指的是：c 源码文件通过 gcc 编译得到可执行文件（elf）。通常使用 gcc 命令 `gcc xxx.c -o xxx` 一步就完成了。但是，这一步里面其实有四个步骤：预处理、编译、汇编、链接。

关于 gcc 命令具体怎么用可以看：[gcc(1) - GNU project C and C++ compiler](https://man7.org/linux/man-pages/man1/gcc.1.html)。这里会用到 `-E`、`-S`、`-c`、`-o` 几个参数。

`-E`：预处理；`-S`：预处理、编译，得到汇编代码；`-c`：预处理、编译、汇编，得到可重定位文件；`-o`：指定目标名称；如果什么参数都不写，那就会预处理、编译、汇编、链接一步到位，得到可执行文件。

- preprocessing（预处理）：`gcc -E xxx.c -o xxx.i`，预处理器会在源码的基础上增加一些代码。
- compilation（编译）：`gcc -S xxx.i -o xxx.s`，编译器通过编译代码（词法分析、语法分析等）得到汇编代码。
- assemble（汇编）：`gcc -c xxx.s -o xxx.o`，汇编器把汇编代码转化为机器指令，这一步得到的是可重定位文件。
- link（链接）：`gcc xxx.o -o xxx`，连接器把各个模块链接起来，组织成为可执行文件。

在链接中，会把函数的名字和变量的名字都称为 symbol（符号）。源代码中的函数的名字和变量的名字，无论长度多少，最终都会转换为符号。

{demo-c}/demo-in-linux/helloworld/ 目录准备了：helloworld.c、helloworld.i、helloworld.s、helloworld.o、helloworld 五个文件。对应上面的源码文件，还有每个命令执行之后生成的文件。

### 看一下这几个文件

#### file

通过 `file` 命令可以查看文件的类型。关于 file 命令具体怎么用可以看：[file(1) - determine file type](https://man7.org/linux/man-pages/man1/file.1.html)。

```
> file helloworld.c
helloworld.c: C source, ASCII text, with CRLF line terminators
> file helloworld.i
helloworld.i: C source, ASCII text
> file helloworld.s
helloworld.s: assembler source, ASCII text
> file helloworld.o
helloworld.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
> file helloworld
helloworld: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=79fbadd19d424598be050dd0fb91297abd895864, for GNU/Linux 3.2.0, not stripped
```

可以看到：helloworld.c 和 helloworld.i 都是 c 源码文件、helloworld.s 是汇编源码文件、helloworld.o 是可重定位文件、helloworld 是可执行文件。

#### objdump

通过 `objdump` 命令可以查看文件的具体内容。关于 objdump 命令具体怎么用可以看：[objdump(1) - display information from object files](https://man7.org/linux/man-pages/man1/objdump.1.html)。

这里会用到 `-h`、`-s`、`-d` 几个参数。大概是：`-h` 输出段表；`-s` 输出段表中每个段的详细内容；`-d` 输出 `.text` 段对应的汇编代码。

这里观察一下 helloworld.o 文件。命令具体的输出放在 {demo-c}/demo-in-linux/helloworld/objdump.md 里面。

命令输出的结果的第一行都是 `helloworld.o:     file format elf64-x86-64`，这个是文件的类型。

`-h` 输出段表：

- `.text` 是代码段，它用于存储程序指令（程序的代码）
- `.data`、`.bss`、`.rodata` 都是数据段，它们用于存储程序数据。
- `.bss` 是未初始化数据段；`.rodata` 是只读数据段。
- 可执行文件 = 程序指令 + 程序数据

`-s` 输出段表中每个段的详细内容，这里截取了 `.text` 段：

```
Contents of section .text:
 0000 f30f1efa 554889e5 488d0500 00000048  ....UH..H......H
 0010 89c7e800 000000b8 00000000 5dc3      ............].
```

每行的第一段是内存偏移量。从第二段开始的一段一段的，表示程序指令的内容（16 进制机器指令）。最后面的那串字符是程序指令的 ASICC 文本。

`-d` 输出 `.text` 段对应的汇编代码，这里截取了 `.text` 段的汇编代码：

```
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

`Disassembly of section .text`，表示程序指令的内容，以及对应的汇编代码。这里和 `-s` 输出的结果对应。上面 `-s` 输出的结果里的 `Contents of section .text` 里的 `f30f1efa`。就对应 `Disassembly of section .text` 里的 `0:	f3 0f 1e fa`。

第二行的 `4:	55` 和后面的 `push   %rbp`，就表示 `55`（0x55）反汇编后对应的汇编指令就是 `push %rbp`。

然后在观察一下 helloworld 文件。命令具体的输出也放在 {demo-c}/demo-in-linux/helloworld/objdump.md 里面。

这里可以看到，helloworld 的内容是比 helloworld.o 要多出不少的。而且通过 helloworld 的内容，我们可以发现 helloworld 的 `.text` 段的第一个函数不是 `main` 函数，而是 `_start` 函数。`_start` 函数会调用 `__libc_start_main` 函数进行一些必要的初始化操作，然后再调用 `main` 函数。
