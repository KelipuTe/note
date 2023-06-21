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

malloc_usable_size()函数返回由malloc()或其他类似内存分配函数分配的内存块的大小。由于内存对齐和其他特定的实现因素，这个大小可能与传递给malloc()的大小不同。

在你提供的代码中，你使用malloc()分配了8个字节的内存来存储字符串 "stringC"。然而，由于内存对齐和其他特定实现因素，malloc()分配的内存块的实际大小可能更大。在你的例子中，似乎malloc()分配的内存块的实际大小是24字节。

这是因为大多数内存分配函数是以块或页为单位分配内存的，通常比请求的大小大。此外，一些内存分配函数可能会给内存块添加额外的元数据，如页眉或页脚，以跟踪内存块的大小和其他信息。

因此，尽管你只请求了8字节的内存，但malloc()分配的内存块的实际大小可能更大，这就是为什么malloc_usable_size()返回24字节。

```text
&strA=0x7fffffffdea0,strA=stringA
&strB=0x55555555601e,strB=stringB
&strB=0x555555556038,strB=stringBB
&strC=0x5555555596b0,strC=stringC
malloc_usable_size(strC)=24
```


```text
(gdb) p strA
$1 = "stringA\000\000\000\000\000\000\000\000"
(gdb) p &strA
$2 = (char (*)[16]) 0x7fffffffdea0
(gdb) x/16xb 0x7fffffffdea0
0x7fffffffdea0:	0x73	0x74	0x72	0x69	0x6e	0x67	0x41	0x00
0x7fffffffdea8:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
```

```
555555556000-555555557000 r--p 00002000 00:25 48                         /mnt/hgfs/demo-c/demo-in-linux/memory/string.elf
555555559000-55555557a000 rw-p 00000000 00:00 0                          [heap]
7ffffffde000-7ffffffff000 rw-p 00000000 00:00 0                          [stack]
```

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

### 数组

一维数组或者多维数组的变量都指向数组所占内存单元的起始地址。

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

```text
555555559000-55555557a000 rw-p 00000000 00:00 0                          [heap]
7ffffffde000-7ffffffff000 rw-p 00000000 00:00 0                          [stack]
```

### 结构体

## 参考

- {51CTO学堂}/{可用行师}/[内存与数据精讲](https://edu.51cto.com/course/29937.html)
- [Bito](https://bito.ai/)
- [DeepL](https://www.deepl.com/translator)