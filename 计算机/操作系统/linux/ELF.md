---
draft: false
date: 2023-06-10 08:00:00 +0800
title: "ELF"
summary: "ELF 文件是什么；ELF 文件的内容是什么；"
toc: true

categories:
  - 操作系统

tags:
  - 计算机
  - 操作系统
  - Linux
---

## 反向链接

[可执行文件](/计算机/程序/可执行文件)；

## 资料

代码：{demo-c}/demo-in-linux/elf/

## 正文

### ELF 文件是什么

C 源码文件通过 gcc 编译器编译可以得到可执行文件。这里的可执行文件就是 ELF 文件的一种。

关于 elf 具体的细节可以看文档 elf(5)。

> elf(5)<br/>
> Amongst these files are normal executable files, relocatable object files, core files, and shared objects.

elf 文件格式有四种。
普通可执行文件（normal executable files）；可重定位文件（relocatable object files）；
核心文件（core files）；共享目标文件、共享库、动态库（shared objects）。

### ELF 文件的内容

这里主要围绕 symbol.c 编译得到的 symbol.elf 文件。

需要用到 readelf 和 objdump 命令。具体用法看文档 objdump(1) 和 readelf(1)。

objdump 主要会用到 "-s"、"-d"、"-M" 几个参数。
"-s" 表示以二进制的形式输出文件内容；"-d" 表示输出汇编代码；

readelf 主要会用到 "-h"、"-l"、"-S"、"-s" 几个参数。
"-h" 表示输出 elf 文件头（elf header）；"-l" 表示输出 program headers；
"-S" 表示输出段表（section headers）；"-s" 表示输出符号表；

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
- Type：elf 文件的类型，这里是可执行文件（Executable file）。
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

### 数据是怎么存储的

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

通过 size 命令可以查看文件中各段及其总和的大小，单位是字节。
关于 size 命令具体怎么用可以看文档 "size(1)"。

这里观察一下 symbol.elf 文件。

```
> size symbol.elf
   text	   data	    bss	    dec	    hex	filename
   1786	    608	     16	   2410	    96a	symbol.elf
```

- text，代码段，通常是指用来存放程序执行代码的一块内存区域。
- data，数据段，通常是指用来存放程序中已初始化的全局变量的一块内存区域。数据段属于静态内存分配。
- bss，通常是指用来存放程序中未初始化的全局变量的一块内存区域。bss 段属于静态内存分配。
- 默认情况下，段的大小是以十进制的方式来展示。

### 程序的入口在哪里

从 `readelf -h` 命令输出的结果里可以知道 symbol.elf 程序的入口地址是 0x1060。
然后在 `objdump -s` 命令输出的结果里面。找到 ".text" 段，然后找到地址 0x1060 对应的数据。
这里截取了 ".text" 段前三行的内容。

```
Contents of section .text:
 1060 f30f1efa 31ed4989 d15e4889 e24883e4  ....1.I..^H..H..
 1070 f0505445 31c031c9 488d3d6c 010000ff  .PTE1.1.H.=l....
 1080 15532f00 00f4662e 0f1f8400 00000000  .S/...f.........
```

- 左边第 1 列是虚拟地址；中间的 4 列是指令码；最右边 1 列是 ASCII 码。
- "0x1060" 是起始地址；"f30f1efa" 是起始指令；指令是 16 进制的："0xf3 0x0f 0x1e 0xfa"；大小为 4 个字节。

在 `objdump -d` 命令输出的结果中，可以找到对应的汇编代码。
这里截取了程序的入口地址对应的部分。通过虚拟地址的值和指令的值可以对应起来。

```
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

"_start" 是函数名。因为 "0x1060" 是起始地址，所以 "_start" 函数，就是这个程序的入口函数。
它在调用 main 函数之前，会做一些前期的准备工作。编程语言的 main 函数一般不是程序的入口函数。
大部分入口函数都是类似 "_start" 函数这样的，而且不同的语言在初始化阶段会有各自的处理逻辑。

这里最右边的汇编代码是 ATT 格式的汇编语法，汇编语法有很多种，ATT 格式主要用于 unix/linux 系统。
使用命令 `objdump -d -M intel symbol.elf` 就可以输出 intel 格式的汇编代码。

### 符号表

通过命令 `readelf -s` 可以输出符号表。
这里截取了 "_start" 对应的数据和 symbol.elf 代码里直接出现的几个。

```
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    12: 0000000000004014     4 OBJECT  LOCAL  DEFAULT   25 staticIntB.1
    13: 0000000000004020     4 OBJECT  LOCAL  DEFAULT   26 staticIntA.0
    26: 000000000000401c     4 OBJECT  GLOBAL DEFAULT   26 globalIntA
    27: 0000000000001149    54 FUNC    GLOBAL DEFAULT   16 functionA
    28: 00000000000011b5    54 FUNC    GLOBAL DEFAULT   16 functionC
    34: 0000000000001060    38 FUNC    GLOBAL DEFAULT   16 _start
    36: 00000000000011eb    75 FUNC    GLOBAL DEFAULT   16 main
    38: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   25 globalIntB
    40: 000000000000117f    54 FUNC    GLOBAL DEFAULT   16 functionB
```

这里用 globalIntB 举例，globalIntB 在源码中是个全局 int 变量，初始化为 300。

- Name（符号名）：globalIntB
- Ndx（段 id）：25，表示符号属于第 25 段。结合段表，第 25 段是 ".data" 段。第 26 段是 ".bss" 段。
- Type（类型）：OBJECT，表示是个对象。如果是 FUNC，就表示是个函数
- Bind（绑定范围）：GLOBAL，表示全局。
- Value（地址）：0x0000000000004010。

这里结合 ".data" 段的数据。
globalIntB 的数据保存在地址从 0x4010 开始往后的 4 个字节上，也就是上面的 "2c010000"，值就是 300。

nm 命令也可以输出符号表。具体怎么用可以看文档 nm(1)。

### 使用符号表的地址直接访问数据

在程序中可以直接使用符号表中的 Value 值去访问对应的内存数据。

同样的源文件，每次编译得到的符号表的地址是一样的。
如果只是修改了某个变量的值，没有大规模的修改代码的话。
重新编译的时候，变量的地址一般也是不会变的。可以通过这种方式去验证。

在 CentOS 7 中，可以直接访问。在 Ubuntu 22.04 中，无法直接访问。不知道什么原因，我也没深究。

### 进程虚拟地址空间映射

需要注意的是，在上面的输出中，与地址有关的数据，都不是程序跑起来的时候，在内存中真正的地址。
程序在进程里跑起来的时候，操作系统会把真正的内存地址和 elf 文件中的虚拟地址做映射。
