---
draft: false
date: 2023-06-10 08:00:00 +0800
title: "ELF 程序入口"
summary: "程序的入口在哪里；"
toc: true

categories:
  - Linux

tags:
  - 计算机科学
  - 操作系统
  - Linux
---

## 反向链接

[ELF](/post/computer-science/operating-system/linux/ELF)；

## 资料

代码：{demo-c}/demo-in-linux/elf/

## 正文

### 程序的入口在哪里

从 `objdump -s` 命令输出的结果里可以知道 symbol.elf 程序的入口地址是 0x1060。
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

这里最右边的汇编代码是 ATT 格式的汇编语法，汇编语法有 intel 格式和 ATT 格式，ATT 格式主要用于 unix/linux 系统。
使用命令 `objdump -d -M intel symbol.elf` 就可以输出 intel 格式的汇编代码。
