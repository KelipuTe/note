---
draft: false
date: 2021-12-06 08:00:00 +0800
lastmod: 2023-02-07 08:00:00 +0800
title: "ELF 文件"
summary: "可执行文件；字节序；符号表；"
toc: true

categories:
- operating-system(操作系统)

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- linux-c
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> VMware Workstation Pro 16<br/>
> Ubuntu 22.04<br/>
> gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0

### 前言

先看：[什么是程序](/post/computer-science/operating-system/linux/program)

### 资料

- [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/elf/

### ELF

之前提到过，c 源码文件通过 gcc 编译得到可执行文件，这里的可执行文件就是 elf 文件的一种。关于 elf 具体的细节可以看 [elf(5) - format of Executable and Linking Format (ELF) files](https://man7.org/linux/man-pages/man5/elf.5.html)。

elf 文件格式有四种：
- normal executable files（普通可执行文件）
- relocatable object files（可重定位文件）
- core files（核心文件）
- shared objects（共享目标文件、共享库、动态库）

#### objdump

这里观察一下 demo02 文件。命令具体的输出放在 {demo-c}/demo-in-linux/elf/objdump.md 里面。

#### readelf

通过 `readelf` 命令可以查看 elf 文件的具体信息。关于 readelf 命令具体怎么用可以看：[readelf(1) - display information about ELF files](https://man7.org/linux/man-pages/man1/readelf.1.html)。

这里会用到 `-h`、`-l`、`-S`、`-s` 几个参数。大概是：`-h` 输出 elf header （elf 文件头）、`-l` 输 program headers、`-S` 输出 section headers（段表）、`-s` 输出符号表。

这里观察一下 demo02 文件。命令具体的输出放在 {demo-c}/demo-in-linux/elf/readelf.md 里面。

`-h` 输出 elf header，这里截取了一些。

```
ELF Header:
 Data:                              2's complement, little endian
 Type:                              DYN (Position-Independent Executable file)
 Machine:                           Advanced Micro Devices X86-64
 Entry point address:               0x1060
 Number of section headers:         31
```

- `Data`：数据存储方式（这里是小端字节序）。
- `Type`：elf 文件的类型。
- `Machine`：机器架构。
- `Entry point address`：程序的入口地址。
- `Number of section headers`：elf 段表大小。

操作系统在加载完可执行文件后，会把控制权转移给该程序，然后找到入口地址（这里就是 `0x1060`）开始运行程序。

`-S` 输出 section headers（段表），这里截取了一些。

```
Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 1] .interp           PROGBITS         0000000000000318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [12] .init             PROGBITS         0000000000001000  00001000
       000000000000001b  0000000000000000  AX       0     0     4
  [16] .text             PROGBITS         0000000000001060  00001060
       0000000000000189  0000000000000000  AX       0     0     16
  [18] .rodata           PROGBITS         0000000000002000  00002000
       0000000000000012  0000000000000000   A       0     0     4
  [25] .data             PROGBITS         0000000000004000  00003000
       0000000000000018  0000000000000000  WA       0     0     8
  [26] .bss              NOBITS           0000000000004018  00003018
       0000000000000010  0000000000000000  WA       0     0     4
  [27] .comment          PROGBITS         0000000000000000  00003018
       000000000000002b  0000000000000001  MS       0     0     1
  [28] .symtab           SYMTAB           0000000000000000  00003048
       00000000000003f0  0000000000000018          29    20     8
  [29] .strtab           STRTAB           0000000000000000  00003438
       00000000000001fa  0000000000000000           0     0     1
```

- `.interp`：程序解释器（elf 解释器）的路径。
- `.init`：进程初始化的代码。
- `.text`：程序指令-->指令码-->机器码-->二进制指令-->cpu 指令（芯片层级的指令）。
- `.rodata`：只读数据（如：数字常量，字符串常量）。
- `.data`：已经给值的全局变量、静态变量、局部变量。
- `.bss`：未初始化的全局变量、静态变量、局部变量。
- `.symtab`：符号表。和`.strtab`一起用。
- `.strtab`：字符串符号表（变量名、函数名）。
- `.debug`：调试信息。
- 其他的段的解释详见 linux 文档。

`-s` 输出符号表，这里截取了 demo02.c 代码里直接出现的几个。

```
Symbol table '.symtab' contains 42 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    12: 0000000000004014     4 OBJECT  LOCAL  DEFAULT   25 si.1
    13: 0000000000004020     4 OBJECT  LOCAL  DEFAULT   26 sj.0
    20: 0000000000001149    54 FUNC    GLOBAL DEFAULT   16 func1
    25: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   25 ga
    32: 000000000000117f    51 FUNC    GLOBAL DEFAULT   16 func2
    36: 00000000000011b2    55 FUNC    GLOBAL DEFAULT   16 main
    37: 000000000000401c     4 OBJECT  GLOBAL DEFAULT   26 gb
```

### 下面一个一个看

#### 数据存储方式

数据存储方式有两种：little endian（小端字节序）和 big endian（大端字节序）。小端字节序又叫主机字节序，大端字节序又叫网络字节序。上面 `readelf -h demo02` 命令的结果里面，在 Data 字段里可以看见 `little endian`，意思就是在 demo02 里面，数据的存储格式是小端字节序。下面来验证一下这个结论。

在 demo02 的源码里面，全局变量 ga 是一开始就明确的给了值的。在 `readelf -s demo02` 命令输出的符号表里，找到 ga 在的那一行，然后就可以知道到 ga 对应的内存地址是 0x4010。然后在 `objdump -s demo02` 命令输出的结果里面，找到 `.data` 段，然后找到地址 0x4010 对应的数据。

这里截取一下 `.data` 段全部的内容。

```
Contents of section .data:
 4000 00000000 00000000 08400000 00000000  .........@......
 4010 2c010000 0a000000                    ,.......    
```

这里通过地址，就可以找到 0x4010 地址上对应的数据是什么。因为源码里面 ga 是 int 类型的，int 类型的长度是 4 个字节，所以 ga 的数据就对应从 0x4010 地址开始的 4 个字节的数据。也就是 `2c010000` 这一段。

源码里面 ga 初始化的时候是 300。10 进制的 300 用 16 进制表示就是 0x12c，补全 4 个字节就是 `00 00 01 2c`。但是，这里可以看到内存上的数据是 `2c 01 00 00`，是反的。这就是因为这里用的是小端字节序。

c 语言的 int 变量由 4 个字节组成，每个字节由 8 个 bit 位组成。把 10 进制的 300 转换成 2 进制就是 `00000000 00000000 00000001 00101100`，左边定义为高位，右边定义为低位。小端字节序的存储格式是把数据的低位放在内存低位上，而内存的排布是从低位到高位的，所以就变成了 `00101100 00000001 00000000 00000000`，转换成 16 进制就是 `2c 01 00 00`。

#### size

通过命令 `size` 可以查看文件中各段及其总和的大小，单位是字节。关于 size 命令具体怎么用可以看：[size(1) - list section sizes and total size of binary files](https://man7.org/linux/man-pages/man1/size.1.html)。

这里观察一下 demo02 文件。

```
> size demo02
   text	   data	    bss	    dec	    hex	filename
   1588	    608	     16	   2212	    8a4	demo02
```

- text，代码段通常是指用来存放程序执行代码的一块内存区域。
- data，数据段通常是指用来存放程序中已初始化的全局变量的一块内存区域。数据段属于静态内存分配。
- bss，bss 段通常是指用来存放程序中未初始化的全局变量的一块内存区域。bss 段属于静态内存分配。
- 默认情况下，段的大小是以十进制的方式来展示。

#### 程序的入口地址

从 `objdump -s demo02` 命令输出的结果里可以知道 demo02 程序的入口地址是 0x1060。然后在 `objdump -s demo02` 命令输出的结果里面。找到 `.text` 段，然后找到地址 0x1060 对应的数据。命令输出的结果在文件 {demo-c}/demo-in-linux/elf/objdump.md 中。这里截取了 `.text` 段全部的内容。

```
Contents of section .text:
 1060 f30f1efa 31ed4989 d15e4889 e24883e4  ....1.I..^H..H..
 1070 f0505445 31c031c9 488d3d33 010000ff  .PTE1.1.H.=3....
 1080 15532f00 00f4662e 0f1f8400 00000000  .S/...f.........
 1090 488d3d81 2f000048 8d057a2f 00004839  H.=./..H..z/..H9
 10a0 f8741548 8b05362f 00004885 c07409ff  .t.H..6/..H..t..
 10b0 e00f1f80 00000000 c30f1f80 00000000  ................
 10c0 488d3d51 2f000048 8d354a2f 00004829  H.=Q/..H.5J/..H)
 10d0 fe4889f0 48c1ee3f 48c1f803 4801c648  .H..H..?H...H..H
 10e0 d1fe7414 488b0505 2f000048 85c07408  ..t.H.../..H..t.
 10f0 ffe0660f 1f440000 c30f1f80 00000000  ..f..D..........
 1100 f30f1efa 803d0d2f 00000075 2b554883  .....=./...u+UH.
 1110 3de22e00 00004889 e5740c48 8b3de62e  =.....H..t.H.=..
 1120 0000e819 ffffffe8 64ffffff c605e52e  ........d.......
 1130 0000015d c30f1f00 c30f1f80 00000000  ...]............
 1140 f30f1efa e977ffff fff30f1e fa554889  .....w.......UH.
 1150 e58b05bd 2e000089 c6488d05 a40e0000  .........H......
 1160 4889c7b8 00000000 e8e3feff ff8b05a1  H...............
 1170 2e000083 c0018905 982e0000 905dc3f3  .............]..
 1180 0f1efa55 4889e548 83ec2089 7dec8b45  ...UH..H.. .}..E
 1190 ec8945fc 8b45fc89 c6488d05 6b0e0000  ..E..E...H..k...
 11a0 4889c7b8 00000000 e8a3feff ff8b45fc  H.............E.
 11b0 c9c3f30f 1efa5548 89e5b800 000000e8  ......UH........
 11c0 85ffffff b8000000 00e87bff ffffbf01  ..........{.....
 11d0 000000e8 a7ffffff bf020000 00e89dff  ................
 11e0 ffffb800 0000005d c3                 .......].   
```

- 左边第 1 列是虚拟地址；中间的 4 列是指令码；最右边 1 列是 ASCII 码。
- `0x1060` 是起始地址；`f30f1efa` 是起始指令；
- 指令是 16 进制的：`0xf3 0x0f 0x1e 0xfa`，大小为 4 个字节。

在命令 `objdump -d demo02` 的结果中，可以找到对应的汇编代码。命令输出的结果在文件 {demo-c}/demo-in-linux/elf/objdump.md 中。这里截取了程序的入口地址对应的部分。通过虚拟地址的值和指令的值可以对应起来。

```
Disassembly of section .text:

0000000000001060 <_start>:
    1060:	f3 0f 1e fa          	endbr64 
    1064:	31 ed                	xor    %ebp,%ebp
    1066:	49 89 d1             	mov    %rdx,%r9
    1069:	5e                   	pop    %rsi
    106a:	48 89 e2             	mov    %rsp,%rdx
    106d:	48 83 e4 f0          	and    $0xfffffffffffffff0,%rsp
    1071:	50                   	push   %rax
    1072:	54                   	push   %rsp
    1073:	45 31 c0             	xor    %r8d,%r8d
    1076:	31 c9                	xor    %ecx,%ecx
    1078:	48 8d 3d 33 01 00 00 	lea    0x133(%rip),%rdi        # 11b2 <main>
    107f:	ff 15 53 2f 00 00    	call   *0x2f53(%rip)        # 3fd8 <__libc_start_main@GLIBC_2.34>
    1085:	f4                   	hlt    
    1086:	66 2e 0f 1f 84 00 00 	cs nopw 0x0(%rax,%rax,1)
    108d:	00 00 00 
```

`_start` 是函数名。因为 `0x1060` 是起始地址，所以 `_start` 函数，就是这个程序的入口函数。它在调用 `main` 函数之前，会做一些前期的准备工作。编程语言的 `main` 函数一般不是程序的入口函数，大部分入口函数都是类似 `_start` 函数这样的，而且不同的语言在初始化阶段会有各自的处理逻辑。

#### 符号表

通过命令 `readelf -s demo02` 可以输出符号表。命令输出的结果在文件 {demo-c}/demo-in-linux/elf/readelf.md 中。这里截取了 `_start` 对应的数据和 demo02.c 代码里直接出现的几个。

```
Symbol table '.symtab' contains 42 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    12: 0000000000004014     4 OBJECT  LOCAL  DEFAULT   25 si.1
    13: 0000000000004020     4 OBJECT  LOCAL  DEFAULT   26 sj.0
    20: 0000000000001149    54 FUNC    GLOBAL DEFAULT   16 func1
    25: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   25 ga
    32: 000000000000117f    51 FUNC    GLOBAL DEFAULT   16 func2
    34: 0000000000001060    38 FUNC    GLOBAL DEFAULT   16 _start
    36: 00000000000011b2    55 FUNC    GLOBAL DEFAULT   16 main
    37: 000000000000401c     4 OBJECT  GLOBAL DEFAULT   26 gb
```

这里用 ga 举例，ga 在源码中是个全局 int 变量，初始化为 300。

- Name（符号名）：ga
- Ndx（段 id）：25，表示符号属于第 25 段。结合段表，第 25 段是 `.data` 段。
- Type（类型）：OBJECT，表示是个对象。如果是 FUNC，就表示是个函数
- Bind（绑定范围）：GLOBAL，表示全局。
- Value（地址）：0x0000000000004010。

这里结合 `.data` 段的数据，ga 的数据保存在地址从 0x4010 开始往后的 4 个字节上，也就是上面的`2c010000`，值就是 300。

#### nm

命令 `nm` 也可以输出符号表。`nm demo02` 命令输出的内容在文件 {demo-c}/demo-in-linux/elf/nm.md 中。关于 gcc 命令具体怎么用可以看：[nm(1) - list symbols from object files](https://man7.org/linux/man-pages/man1/nm.1.html)。

#### 使用符号表的地址直接访问数据

在程序中可以直接使用符号表中的 Value 值去访问对应的内存数据。同样的源文件，每次编译得到的符号表的地址是一样的。如果只是修改了某个变量的值，没有大规模的修改代码的话，重新编译的时候，变量的地址一般也是不会变得。可以通过这种方式验证这个结论。

#### 进程虚拟地址空间映射

在上面的输出中，与地址有关的数据，都不是程序跑起来的时候，在内存中真正的地址。程序在进程里跑起来的时候，操作系统会把真正的内存地址和 elf 文件中的虚拟地址做映射。
