---
draft: false
date: 2022-06-18 08:00:00 +0800
title: "指令和数据"
summary: "指令；数据；栈和堆；指针；字符串；数组；结构体；"
toc: true

categories:
  - operating-system

tags:
  - computer-science
  - operating-system
  - memory
---

## 前言

## 正文

这里就麻烦一点了 gdb 调试

### 内存布局

内存的存储单元会根据 CPU 位数宽度以 16 进制从 0 开始顺序编号。每个存储单元只对应一个编号，且只可以存储一个 byte 的数据。

Linux 进程的虚拟内存会对内存空间进行划分，不同的区域存储不同的数据，有不同的权限。

一般来说：可执行的指令存储在代码区 .text、全局变量和静态变量存储在数据区 .data，局部变量存储在 stack 区（栈区），动态申请的内存存储在
heap 区（堆区）。

- statically allocated（静态分配）：data on the stack（数据在栈区）
- dynamically allocated（动态分配）：data on the heap（数据在堆区）

可执行文件运行的时候，代码和数据会被读取到内存中。

### linux 进程的内存布局

- 内核空间（kernel space）
- 栈（stack）
- 动态库的映射
- 堆（heap）
- 读写数据区，主要是程序数据，`.data`、`.bss` 等
- 只读数据区，主要是程序指令，`.text`、`.init`、`.rodata` 等
- 保留区

### 起始地址（首地址、基地址、基址）

起始地址指的是函数代码或者变量所占内存空间的第一个存储单元的地址。

### 指令

这里的指令说的是代码块（函数）。函数的起始地址在编译时就已经确定了。

readelf -s

```text
Symbol table '.symtab' contains 37 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    23: 0000000000001149    15 FUNC    GLOBAL DEFAULT   16 methodA
```

objdump -s

```text
Contents of section .text:
 1140 ________ ________ __f30f1e fa554889  .....w.......UH.
 1150 e5b80004 00005dc3 ________ ________  ......].....UH..
```

objdump -d

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

起始地址也占了一个位置，所以计算结束地址的时候不是直接加函数大小。

起始地址为 0x1149，结束地址为 0x1157 = 起始地址（16） + 15（10）
49,4a,4b,4c,4d, 4e,4f,50,51,52, 53,54,55,56,57,

gdb 调试

```text
(gdb) p methodA
$1 = {int ()} 0x555555555149 <methodA>

(gdb) x/15xb 0x555555555149
0x555555555149 <methodA>:	0xf3	0x0f	0x1e	0xfa	0x55	0x48	0x89	0xe5
0x555555555151 <methodA+8>:	0xb8	0x00	0x04	0x00	0x00	0x5d	0xc3
```

在程序里一个字节一个字节的打印函数内存地址上的内容

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

前面不是有三个字节的空位吗，这个时候会扩展符号位以填充前面的空位，这样，就得到了 0xffffffc3。

### 数据

全局变量的起始地址在编译时就已经确定了。

readelf -s

```text
Symbol table '.symtab' contains 37 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    27: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   25 globalI
```

objdump -s

```text
Contents of section .data:
 4000 00000000 00000000 08400000 00000000  .........@......
 4010 00040000 
```

gdb 调试

```text
(gdb) p &globalI
$3 = (int *) 0x555555558010 <globalI>

(gdb) x/4xb 0x555555558010
0x555555558010 <globalI>:	0x00	0x04	0x00	0x00
```

注意，在内存中是小端字节序。

按字节输出 globalI，globalI 取地址，拿到的是 int* ，对 int* +1 会移动4个字节，这不是我们想要的

如果想按字节输出，首先需要将 地址强转陈 chat* ，chat* +1 移动的就是1个字节了

### 有意思的来了

readelf -s

```text
Symbol table '.symtab' contains 38 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
    23: 0000000000001149    16 FUNC    GLOBAL DEFAULT   16 methodA
    28: 0000000000004010     4 OBJECT  GLOBAL DEFAULT   25 globalI
```

objdump -d

```text
0000000000001149 <methodA>:
    1149:	f3 0f 1e fa          	endbr64 
    114d:	55                   	push   %rbp
    114e:	48 89 e5             	mov    %rsp,%rbp
    1151:	8b 05 b9 2e 00 00    	mov    0x2eb9(%rip),%eax        # 4010 <globalI>
    1157:	5d                   	pop    %rbp
    1158:	c3                   	ret
```

globalI 是全局变量，methodA 返回了全局变量，在汇编里面可以看到，这里 methodA 直接就用了 globalI 的地址

### 栈

局部变量是分配在栈上的，每次执行的时候都不一样。可以和进程内存信息比对一下，局部变量的地址都在 stack 区（栈区）。

```text
(gdb) p &localI
$1 = (int *) 0x7fffffffdeb4
(gdb) x/4xb 0x7fffffffdeb4
0x7fffffffdeb4:	0x00	0x04	0x00	0x00
```

```text
7ffffffde000-7ffffffff000 rw-p 00000000 00:00 0                          [stack]
```

| -     | 地址             |
|-------|----------------|
| 栈堆首地址 | 0x7ffffffde000 |
| 变量首地址 | 0x7fffffffdeb4 |
| 栈堆尾地址 | 0x7ffffffff000 |

### 堆

```text
(gdb) p p7LocalJ
$1 = (int *) 0x5555555596b0
(gdb) x/4xb 0x5555555596b0
0x5555555596b0:	0x00	0x08	0x00	0x00
```

```text
555555559000-55555557a000 rw-p 00000000 00:00 0                          [heap]
```

| -    | 地址             |
|------|----------------|
| 堆首地址 | 0x555555559000 |
| 变量地址 | 0x5555555596b0 |
| 堆尾地址 | 0x55555557a000 |

### 指针

```text
(gdb) p &localI
$2 = (int *) 0x7fffffffdea4
(gdb) x/4xb 0x7fffffffdea4
0x7fffffffdea4:	0x00	0x04	0x00	0x00
```

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

### 字符串

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

stringB 和 stringBB 并没有开辟内存去存它们，程序运行起来之后，也没有办法去修改它们的值。所以，在编译的时候，就放到了只读数据段，也就是 ".rodata" 段。

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
