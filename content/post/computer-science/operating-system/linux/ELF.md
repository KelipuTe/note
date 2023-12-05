---
draft: false
date: 2023-06-10 08:00:00 +0800
title: "ELF"
summary: "ELF 文件是什么；ELF 文件的内容是什么；"
toc: true

categories:
  - Linux

tags:
  - 计算机科学
  - 操作系统
  - Linux
---

## 反向链接

[程序](/post/computer-science/program/程序)；
[可执行文件](/post/computer-science/program/可执行文件)；

## 资料

代码：{demo-c}/demo-in-linux/elf/

## 正文

### ELF 文件是什么

在笔记 <<程序>> 中提到过，C 源码文件通过 gcc 编译器编译可以得到可执行文件。
这里的可执行文件就是 ELF 文件的一种。

关于 elf 具体的细节可以看 "elf(5) - format of Executable and Linking Format (ELF) files"

> elf(5)<br/>
> Amongst these files are normal executable files, relocatable object files, core files, and shared objects.

elf 文件格式有四种：
普通可执行文件（normal executable files）；
可重定位文件（relocatable object files）；
核心文件（core files）；
共享目标文件、共享库、动态库（shared objects）。

我在 {demo-c}/demo-in-linux/elf/ 目录下，准备了三个程序：
symbol.c、0b_0_10_0x.c、address.c。分别用于讨论不同的问题。

### ELF 文件的内容

这里主要围绕 symbol.c 编译得到的 symbol.elf 文件。

先用 objdump 命令观察一下。
命令具体的输出放在 {demo-c}/demo-in-linux/elf/symbol_elf_objdump.md 里面。

然后，用 readelf 命令观察一下 symbol.elf。
关于 readelf 命令具体怎么用可以看 "readelf(1) - display information about ELF files"。

这里会用到 "-h"、"-l"、"-S"、"-s" 几个参数。
"-h" 表示输出 elf 文件头（elf header）；"-l" 表示输出 program headers；
"-S" 表示输出段表（section headers）；"-s" 表示输出符号表。
命令具体的输出放在 {demo-c}/demo-in-linux/elf/symbol_elf_readelf.md 里面。

> elf(5)<br/>
> The ELF header is always at offset zero of the file.

elf 文件头总是在文件的最前面，其他的数据都在后面。
这很好理解，如果一个文件最前面不告我它是什么，我怎么知道它是什么。

分别解释一下。

"-h" 输出的 elf 文件头，这里截取了一些。

```
  Data:                              2's complement, little endian
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Entry point address:               0x1060
  Number of section headers:         31
```

- Data：数据存储方式，这里是小端字节序（little endian）。
- Type：elf 文件的类型，这里是可执行文件（Executable）。
- Machine：机器架构，这里是 X86-64。
- Entry point address：程序的入口地址。
  操作系统在加载完可执行文件后，会把控制权转移给该程序，然后找到入口地址（这里就是 "0x1060"）开始运行程序。
- Number of section headers：elf 段表的大小。

"-S" 输出 section headers（段表），这里截取了一些。

```
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 1] .interp           PROGBITS         0000000000000318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [12] .init             PROGBITS         0000000000001000  00001000
       000000000000001b  0000000000000000  AX       0     0     4
  [16] .text             PROGBITS         0000000000001060  00001060
       00000000000001d6  0000000000000000  AX       0     0     16
  [18] .rodata           PROGBITS         0000000000002000  00002000
       0000000000000063  0000000000000000   A       0     0     8
  [25] .data             PROGBITS         0000000000004000  00003000
       0000000000000018  0000000000000000  WA       0     0     8
  [26] .bss              NOBITS           0000000000004018  00003018
       0000000000000010  0000000000000000  WA       0     0     4
  [28] .symtab           SYMTAB           0000000000000000  00003048
       0000000000000408  0000000000000018          29    20     8
  [29] .strtab           STRTAB           0000000000000000  00003450
       000000000000022c  0000000000000000           0     0     1
```

- ".interp"：程序解释器（elf 解释器）的路径。
- ".init"：进程初始化的代码。
- ".text"：程序指令-->指令码-->机器码-->二进制指令-->cpu 指令（芯片层级的指令）。
- ".rodata"：只读数据（如：数字常量，字符串常量）。
- ".data"：已经给值的全局变量、静态变量、局部变量。
- ".bss"：未初始化的全局变量、静态变量、局部变量。
- ".symtab"：符号表。和 ".strtab" 段一起用。
- ".strtab"：字符串符号表（变量名、函数名）。
- ".debug"：调试信息，编译的时候带 "-g" 参数的时候就会有这一段。
- 其他的段的解释详见 linux 文档 elf(5)。

"-s" 输出的符号表，这里截取了 symbol.c 代码里直接出现的几个。

```
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    12: 0000000000004014     4 OBJECT  LOCAL  DEFAULT   25 staticIntB.1
    13: 0000000000004020     4 OBJECT  LOCAL  DEFAULT   26 staticIntA.0
    26: 000000000000401c     4 OBJECT  GLOBAL DEFAULT   26 globalIntA
    27: 0000000000001149    54 FUNC    GLOBAL DEFAULT   16 functionA
    28: 00000000000011b5    54 FUNC    GLOBAL DEFAULT   16 functionC
    36: 00000000000011eb    75 FUNC    GLOBAL DEFAULT   16 main
    38: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   25 globalIntB
    40: 000000000000117f    54 FUNC    GLOBAL DEFAULT   16 functionB
```

### 数据存储方式

数据存储方式有两种：小端字节序（little endian）和大端字节序（big endian）。
小端字节序又叫主机字节序，大端字节序又叫网络字节序。

上面 `readelf -h` 命令的结果里面，在 Data 字段里可以看见 "little endian"。
意思就是在 symbol.elf 里面，数据的存储格式是小端字节序。下面来验证一下这个结论。

在 symbol.c 的源码里面，全局变量 globalIntB 是一开始就明确的给了值的。
在 `readelf -s` 命令输出的符号表里，找到 globalIntB 在的那一行。
然后，就可以知道到 globalIntB 对应的内存地址是 0x4010。

在 `objdump -s` 命令输出的结果里面，找到内存地址 0x4010，地址上存储的就是全局变量 globalIntB 的数据。
内存地址 0x4010 在 ".data" 段里面，这里截取一下 ".data" 段的内容。

```
Contents of section .data:
 4000 00000000 00000000 08400000 00000000  .........@......
 4010 2c010000 0a000000                    ,.......        
```

因为源码里面 globalIntB 是 int 类型的，int 类型的长度是 4 个字节。
所以，globalIntB 的数据就对应从 0x4010 地址开始的 4 个字节的数据。也就是 "2c010000" 这一段。

源码里面 globalIntB 初始化的时候是 300。
10 进制的 300 用 16 进制表示就是 0x12c，补全 4 个字节就是 "00 00 01 2c"。
但是，这里可以看到内存上的数据是 "2c 01 00 00"，是反的。这就是因为这里用的是小端字节序。

c 语言的 int 变量由 4 个字节组成，每个字节由 8 个 bit 位组成。
把 10 进制的 300 转换成 2 进制就是 "00000000 00000000 00000001 00101100"，左边定义为高位，右边定义为低位。
小端字节序的存储格式是把数据的低位放在内存低位上，而内存的排布是从低位到高位的。
所以，就变成了 "00101100 00000001 00000000 00000000"，转换成 16 进制就是 "2c 01 00 00"。

关于进制的问题，我放了一个示例代码，0b_0_10_0x.c。

通过 size 命令可以查看文件中各段及其总和的大小，单位是字节。
关于 size 命令具体怎么用可以看 "size(1) - list section sizes and total size of binary files"。

这里观察一下 symbol.elf 文件。

```
> size demo02
   text	   data	    bss	    dec	    hex	filename
   1786	    608	     16	   2410	    96a	symbol.elf
```

- text，代码段，通常是指用来存放程序执行代码的一块内存区域。
- data，数据段，通常是指用来存放程序中已初始化的全局变量的一块内存区域。数据段属于静态内存分配。
- bss，通常是指用来存放程序中未初始化的全局变量的一块内存区域。bss 段属于静态内存分配。
- 默认情况下，段的大小是以十进制的方式来展示。

## 正向链接

[ELF_数据存储方式](/post/computer-science/operating-system/linux/ELF_数据存储方式)；
[ELF_程序入口](/post/computer-science/operating-system/linux/ELF_程序入口)；
[ELF_符号表](/post/computer-science/operating-system/linux/ELF_符号表)；
