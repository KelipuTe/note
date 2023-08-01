---
draft: false
date: 2023-06-28 08:00:00 +0800
title: "内存中的指令和数据"
summary: "指令；数据；栈和堆；指针；字符串；数组；结构体；"
toc: true

categories:
  - operating-system(操作系统)

tags:
  - computer-science(计算机科学)
  - operating-system(操作系统)
  - memory(内存)
---

## 前言

这篇笔记需要 elf、linux 进程的内存布局、gdb 调试相关的知识。

## 资料

笔记里的代码都在 [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/memory/ 目录下。

图在 <a href="/drawio/computer-science/operating-system/memory/memory.drawio.html">memory.drawio.html</a> 里面。

## 正文

### 内存地址

内存的存储单元会根据 CPU 的位宽以 16 进制从 0 开始顺序编号。
每个存储单元只对应一个编号，且只可以存储一个 byte 的数据。

32 位 CPU 最大的内存地址是 0xffffffff，容量为 2^32 字节
64 位 CPU 最大的内存地址是 0xffffffffffffffff，容量为 2^64 字节。

### 起始地址

起始地址（首地址、基地址、基址），指的是代码块（函数）或者数据（变量）所占内存空间的第一个存储单元的地址。

### 指令

这里的指令说的是代码块（函数）。函数的起始地址在编译时就已经确定了。

代码：instruction_in_memory.c

从 `readelf -s` 的结果里，找到代码块（methodA）的信息。
这里可以看到，代码块的起始地址是 0x1149，代码块的长度是 15。

```text
Symbol table '.symtab' contains 37 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    23: 0000000000001149    15 FUNC    GLOBAL DEFAULT   16 methodA
```

然后，在 `objdump -s` 的结果里，找到起始地址的位置。
从起始地址开始的 15 个字节就是代码块对应的数据了。

```text
Contents of section .text:
 1140 ________ ________ __f30f1e fa554889  .....w.......UH.
 1150 e5b80004 00005dc3 ________ ________  ......].....UH..
```

起始地址也占了一个位置，所以计算结束地址的时候不是直接加代码块的大小。
起始地址为 0x1149，结束地址为 0x1157 = 起始地址（16） + (15-1)（10）

也就是 0x1149,0x114a,0x114b,0x114c,0x114d,0x114e,0x114f,0x1150,0x1151,0x1152,0x1153,0x1154,0x1155,0x1156,0x1157，
这 15 个地址上的数据。

可以再看一眼汇编代码，在 `objdump -d` 的结果里，找到起始地址的位置。
这里的数据和上面的 15 个字节的数据是一样的，这里详细的标注了每一个字节对应的汇编代码。

```text
Disassembly of section .text:

0000000000001149 <methodA>:
    1149:	f3 0f 1e fa          	endbr64 
    114d:	55                   	push   %rbp
    114e:	48 89 e5             	mov    %rsp,%rbp
    1151:	b8 00 04 00 00       	mov    $0x400,%eax
    1156:	5d                   	pop    %rbp
    1157:	c3                   	ret
```

#### gdb 调试

先打印一下代码块在程序中的起始地址，然后，打印从起始地址开始的 15 个地址上的数据。和上面一样的对吧。

```text
(gdb) p methodA
$1 = {int ()} 0x555555555149 <methodA>

(gdb) x/15xb 0x555555555149
0x555555555149 <methodA>:	0xf3	0x0f	0x1e	0xfa	0x55	0x48	0x89	0xe5
0x555555555151 <methodA+8>:	0xb8	0x00	0x04	0x00	0x00	0x5d	0xc3
```

通过遍历也可以在程序里一个字节一个字节的打印代码块对应的内存地址上的内容。

```text
&methodA=0x56025d439149
[0x56025d439149]=fffffff3
[0x56025d43914a]=0f
[0x56025d43914b]=1e
[0x56025d43914c]=fffffffa
[0x56025d43914d]=55
[0x56025d43914e]=48
[0x56025d43914f]=ffffff89
[0x56025d439150]=ffffffe5
[0x56025d439151]=ffffffb8
[0x56025d439152]=00
[0x56025d439153]=04
[0x56025d439154]=00
[0x56025d439155]=00
[0x56025d439156]=5d
[0x56025d439157]=ffffffc3
```

这里打印 f3 出来 fffffff3 是因为符号扩展（sign extension），这里简单解释一下。
当 0xc3 被存储在 1 个字节的 char 变量中，被提升为 4 个字节的 int 或者被以十六进制打印时。
前面不是有三个字节的空位嘛，这个时候会扩展符号位以填充前面的空位，这样，就得到了 0xffffffc3。

### 数据

全局变量的起始地址在编译时就已经确定了。

代码：data_in_memory.c

从 `readelf -s` 的结果里，找到数据（globalI）的信息。
这里可以看到，数据的起始地址是 0x4010，数据的长度是 4。

```text
Symbol table '.symtab' contains 37 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    27: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   25 globalI
```

然后，在 `objdump -s` 的结果里，找到起始地址的位置。
从起始地址开始的 4 个字节就是代码块对应的数据了。

```text
Contents of section .data:
 4000 00000000 00000000 08400000 00000000  .........@......
 4010 00040000 
```

起始地址为 0x4010，结束地址为 0x4013 = 起始地址（16） + 3（10）

也就是 0x4010,0x4011,0x4012,0x4013 这 4 个地址上的数据。

#### gdb 调试

这里和指令那里一样，先打印数据的起始地址，然后，打印地址上的数据。注意，数据在内存中是小端字节序。

```text
(gdb) p &globalI
$3 = (int *) 0x555555558010 <globalI>

(gdb) x/4xb 0x555555558010
0x555555558010 <globalI>:	0x00	0x04	0x00	0x00
```

和指令那里一样，也通过遍历也可以在程序里一个字节一个字节的打印数据对应的内存地址上的内容。

但是，要注意。对 int 类型进行取地址操作，拿到的是 int*。对 int* +1 会移动4个字节，这不是我们想要的。
如果想输出 int 类型的每个字节，首先需要将起始地址转换成 chat*，chat* +1 移动的就是 1 个字节了。

### 有意思的来了

如果在代码块里直接返回全局变量会怎么样呢。

代码：return_global.c

从 `readelf -s` 的结果里，找到代码块（methodA）和数据（globalI）的信息。

```text
Symbol table '.symtab' contains 38 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    23: 0000000000001149    16 FUNC    GLOBAL DEFAULT   16 methodA
    28: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   25 globalI
```

然后，我们看汇编代码，在 `objdump -d` 的结果里，找到代码块对应的汇编代码。

```text
0000000000001149 <methodA>:
    1149:	f3 0f 1e fa          	endbr64 
    114d:	55                   	push   %rbp
    114e:	48 89 e5             	mov    %rsp,%rbp
    1151:	8b 05 b9 2e 00 00    	mov    0x2eb9(%rip),%eax        # 4010 <globalI>
    1157:	5d                   	pop    %rbp
    1158:	c3                   	ret
```

globalI 是全局变量，methodA 的代码里直接返回了全局变量，这里的汇编代码，methodA 直接就用了 globalI 的地址。

### 栈

代码：stack.c

静态分配的内存在栈区，比如，函数里面的局部变量，每次执行的时候都不一样。

```text
(gdb) p &localI
$1 = (int *) 0x7fffffffdeb4

(gdb) x/4xb 0x7fffffffdeb4
0x7fffffffdeb4:	0x00	0x04	0x00	0x00
```

```text
7ffffffde000-7ffffffff000 rw-p 00000000 00:00 0                          [stack]
```

可以和进程的内存信息比对一下。
0x7ffffffde000（栈堆首地址） < 0x7fffffffdeb4（变量首地址） < 0x7ffffffff000（栈堆尾地址）

### 堆

代码：heap.c

动态分配的内存在堆区，比如，给指针申请一块内存。

```text
(gdb) p p7LocalJ
$1 = (int *) 0x5555555596b0

(gdb) x/4xb 0x5555555596b0
0x5555555596b0:	0x00	0x08	0x00	0x00
```

```text
555555559000-55555557a000 rw-p 00000000 00:00 0                          [heap]
```

可以和进程的内存信息比对一下。
0x555555559000（堆首地址） < 0x5555555596b0（变量地址） < 0x55555557a000（堆尾地址）

### 指针

代码：pointer.c

首先定义一个变量，比如，定义一个 int 类型的。


```text
(gdb) p &localI
$2 = (int *) 0x7fffffffdea4

(gdb) x/4xb 0x7fffffffdea4
0x7fffffffdea4:	0x00	0x04	0x00	0x00
```

二级指针

```text
(gdb) p p7LocalI
$4 = (int *) 0x7fffffffdea4

(gdb) p &p7LocalI
$3 = (int **) 0x7fffffffdea8

(gdb) x/8xb 0x7fffffffdea8
0x7fffffffdea8:	0xa4	0xde	0xff	0xff	0xff	0x7f	0x00	0x00
```

```text
(gdb) p p7LocalII
$5 = (int **) 0x7fffffffdea8
(gdb) p &p7LocalII
$6 = (int ***) 0x7fffffffdeb0
(gdb) x/8xb &p7LocalII
0x7fffffffdeb0:	0xa8	0xde	0xff	0xff	0xff	0x7f	0x00	0x00
```

我画了个图，**memory.drawio.html 2-2、代码的运行结果在内存里的结构，以及得到结果的过程**

### 字符串

代码：string.c

```text
&strA=0x7fffffffdea0,strA=stringA
&strB=0x55555555601e,strB=stringB
&strB=0x555555556038,strB=stringBB
&strC=0x5555555596b0,strC=stringC
```

打印几个指针指向的地址，然后，结合 "proc/{pid}/maps" 看一下内存。

```
555555556000-555555557000 r--p 00002000 00:25 48                         /mnt/hgfs/demo-c/demo-in-linux/memory/string.elf
555555559000-55555557a000 rw-p 00000000 00:00 0                          [heap]
7ffffffde000-7ffffffff000 rw-p 00000000 00:00 0                          [stack]
```

strA 字符数组地址指向了栈，strB 指针两次都指向了 string.elf 文件，strC 字符数组地址指向了堆。

这里，因为是字符串，所以，可以直接在 `objdump -s` 命令输出的结果里面查找这几个字符串值。
这里直接说结论了，stringA 和 stringC 在 ".text" 段里面，stringB 和 stringBB 在 ".rodata" 段里面。
此外，在 ".rodata" 段里面，还可以看到很多写死在程序里的字符串。比如, 写在 printf() 里面的字符串。

stringA 和 stringC 这两个字符串只是初始化的时候长这样，底下真正对应的字符数组是可变的。

stringA 是一个字符数组的值，这个字符数组，存储在栈里面，编译的时候就知道有多大了。
存储在栈里面的字符数组是可变的，stringA 只用于初始化。

```text
(gdb) p strA
$1 = "stringA\000\000\000\000\000\000\000\000"
(gdb) p &strA
$2 = (char (*)[16]) 0x7fffffffdea0
(gdb) x/16xb 0x7fffffffdea0
0x7fffffffdea0:	0x73	0x74	0x72	0x69	0x6e	0x67	0x41	0x00
0x7fffffffdea8:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
```

同理 stringC 是一个字符数组的值，这个字符数组是程序运行的时候动态申请的内存，动态申请的内存在堆里面。
存储在堆里面的字符数组是可变的，stringC 只用于初始化。

stringB 和 stringBB
并没有开辟内存去存它们，程序运行起来之后，也没有办法去修改它们的值。所以，在编译的时候，就放到了只读数据段，也就是 ".rodata"
段。

```
Contents of section .rodata:
 2000 01000200 7069643d 25640a00 26737472  ....pid=%d..&str
 2010 413d2570 2c737472 413d2573 0a007374  A=%p,strA=%s..st
 2020 72696e67 42002673 7472423d 25702c73  ringB.&strB=%p,s
 2030 7472423d 25730a00 73747269 6e674242  trB=%s..stringBB
 2040 00267374 72433d25 702c7374 72433d25  .&strC=%p,strC=%
 2050 730a006d 616c6c6f 635f7573 61626c65  s..malloc_usable
 2060 5f73697a 65287374 7243293d 256c640a  _size(strC)=%ld.
 2070 00 
```

```text
malloc_usable_size(strC)=24
```

malloc_usable_size() 返回由 malloc() 或其他内存分配函数分配的内存块的大小。
由于内存对齐或者其他的因素，这个大小可能与传递给 malloc() 或其他内存分配函数的大小不同。

在示例代码 string.c 中使用 malloc() 分配了 8 个字节的内存来存储字符串 "stringC"。
但是，这里用 malloc_usable_size() 可以知道，malloc() 分配的内存块的实际大小是 24 字节。

这是因为，大多数内存分配函数是以内存块或内存页为单位进行分配的，通常比请求的大小大。
编译器也有可能会做一些优化，申请内存的时候多给的部分，在需要扩容的时候就可以直接用了。
不需要再通过系统调用去申请，可以减少系统调用的次数。

此外，一些内存分配函数可能会给内存块添加额外的元数据，以跟踪内存块的信息。
比如，调用 free() 函数的时候，只传了一个内存地址过去，它是怎么知道应该释放多大的内存的。

### 数组

代码：array.c

一维数组或者多维数组的变量都指向数组所占内存单元的起始地址。

```text
555555559000-55555557a000 rw-p 00000000 00:00 0                          [heap]
7ffffffde000-7ffffffff000 rw-p 00000000 00:00 0                          [stack]
```

栈和堆就没啥好说的，直接声明的数组在栈里面，动态申请内存的数组是指针，指向堆。

```text
(gdb) p &a5I
$2 = (int (*)[4]) 0x7fffffffde90
(gdb) x/16xb 0x7fffffffde90
0x7fffffffde90:	0x02	0x00	0x00	0x00	0x04	0x00	0x00	0x00
0x7fffffffde98:	0x08	0x00	0x00	0x00	0x10	0x00	0x00	0x00
```

```text
(gdb) p &a5J
$3 = (int (*)[2][2]) 0x7fffffffdea0
(gdb) x/16xb 0x7fffffffdea0
0x7fffffffdea0:	0x20	0x00	0x00	0x00	0x40	0x00	0x00	0x00
0x7fffffffdea8:	0x80	0x00	0x00	0x00	0x00	0x01	0x00	0x00
```

通过观察内存可以发现，不管是 `int[4]` 还是 `int[2][2]`，在内存里都是连续的 16 个字节。

```text
(gdb) p p7a5K
$4 = (int (*)[4]) 0x5555555596b0
(gdb) x/16xb 0x5555555596b0
0x5555555596b0:	0x02	0x00	0x00	0x00	0x04	0x00	0x00	0x00
0x5555555596b8:	0x08	0x00	0x00	0x00	0x10	0x00	0x00	0x00
```

```text
(gdb) p p7a5L
$5 = (int (*)[2][2]) 0x5555555596d0
(gdb) x/16xb 0x5555555596d0
0x5555555596d0:	0x20	0x00	0x00	0x00	0x40	0x00	0x00	0x00
0x5555555596d8:	0x80	0x00	0x00	0x00	0x00	0x01	0x00	0x00
```

运行的时候动态分配的内存也是一样的，都是连续的 16 个字节。

```text
(*p7a5L)[0]=0x56247f1946d0,p7l=0x56247f1946d0
(*p7a5L)[0][0]=32,&(*p7a5L)[0][0]=0x56247f1946d0,p7ll+0=32,p7ll+0=0x56247f1946d0
(*p7a5L)[0][0]=64,&(*p7a5L)[0][0]=0x56247f1946d4,p7ll+0=33,p7ll+0=0x56247f1946d4
(*p7a5L)[1]=0x56247f1946d8,p7l+1=0x56247f1946d8
(*p7a5L)[0][0]=128,&(*p7a5L)[0][0]=0x56247f1946d8,p7ll+0=128,p7ll+0=0x56247f1946d8
(*p7a5L)[0][0]=128,&(*p7a5L)[0][0]=0x56247f1946dc,p7ll+0=256,p7ll+0=0x56247f1946dc
```

指向数组的指针这里，需要关注的就不是内存了，而是指针移动的距离。
当指针声明为 `(int(*)[2])` 的时候，指针 +1，移动的是 8 个字节，也就是一个 `int[2]` 的长度。
当指针声明为 `(int *)` 的时候，指针 +1，移动的是 4 个字节，也就是一个 int 的长度。

### 结构体

代码：struct.c

```text
555555559000-55555557a000 rw-p 00000000 00:00 0                          [heap]
7ffffffde000-7ffffffff000 rw-p 00000000 00:00 0                          [stack]
```

栈和堆就没啥好说的，直接声明的结构体在栈里面，动态申请内存的结构体是指针，指向堆。
两个结构体里的 name 都是动态申请的内存，都指向堆。

```text
sizeof(s6a)=16
&s6a=0x7fffffffdeb0
&s6a.name=0x7fffffffdeb0
s6a.name=0x5555555596b0
&s6a.age=0x7fffffffdeb8
&s6a.sex=0x7fffffffdebc
```

这里用 sizeof() 得出的结构体的大小是 16 个字节。但是，实际上加起来应该是 13 个字节。
声明的时候，结构体里面有三个字段，一个 "char *" 8 个字节，一个 int 4 个字节，一个 char 1 个字节。

这是因为编译器会使用填充和对齐来优化内存的使用和访问。
这会导致同一个 struct 在不同系统上的大小可能不同，这取决于不同系统上的数据类型的大小和对齐的规则。

在这里的情况是，struct 的大小是其最大字段的大小的倍数。
最大的字段是 "char *" 8 个字节，结构体一共需要 13 个字节。所以，分配给结构体的内存就是 16 个字节。

```text
(gdb) x/16xb 0x7fffffffdeb0
0x7fffffffdeb0:	0xb0	0x96	0x55	0x55	0x55	0x55	0x00	0x00
0x7fffffffdeb8:	0x12	0x00	0x00	0x00	0x66	0x00	0x00	0x00
(gdb) x/16xb 0x5555555596b0
0x5555555596b0:	0x61	0x61	0x61	0x00	0x00	0x00	0x00	0x00
0x5555555596b8:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
```

直接观察内存里从结构体首地址开始的 16 个字节。

最前面的 8 个字节，是小端字节序的，正过来就是 00 00 55 55 55 55 96 b0。
这就是 name 字段的地址 0x5555555596b0，前面的 0 被省略了。

然后是 4 个字节的 int。然后是 1 个字节的 char。最后 3 个字节填充的 0 用于对齐。

0x5555555596b0 上的 0x61 就是 a 的 ascii 码（10 进制是 97）。

```text
sizeof(p7b)=8
p7b=0x5555555596d0
&p7b->name=0x5555555596d0
p7b->name=0x5555555596f0
&p7b->age=0x5555555596d8
&p7b->sex=0x5555555596dc
```

这里因为是指针，所以 sizeof() 得出的大小是 8 个字节。

```text
(gdb) x/16xb 0x5555555596d0
0x5555555596d0:	0xf0	0x96	0x55	0x55	0x55	0x55	0x00	0x00
0x5555555596d8:	0x12	0x00	0x00	0x00	0x66	0x00	0x00	0x00
(gdb) x/16xb 0x5555555596f0
0x5555555596f0:	0x62	0x62	0x62	0x00	0x00	0x00	0x00	0x00
0x5555555596f8:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
```

和上面一样，直接观察内存里结构体指针指向的内存地址开始的 16 个字节。

最前面的 8 个字节，是小端字节序的，正过来就是 00 00 55 55 55 55 96 f0。
这就是 name 字段的地址 0x5555555596f0，前面的 0 被省略了。

然后是 4 个字节的 int。然后是 1 个字节的 char。最后 3 个字节填充的 0 用于对齐。

0x5555555596f0 上的 0x62 就是 b 的 ascii 码（10 进制是 98）。
