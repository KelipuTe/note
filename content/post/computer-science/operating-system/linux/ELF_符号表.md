---
draft: false
date: 2023-06-10 08:00:00 +0800
title: "ELF 符号表"
summary: "ELF 符号表；"
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

命令 nm 也可以输出符号表。关于 gcc 命令具体怎么用可以看 "nm(1) - list symbols from object files"。
nm 命令输出的内容在文件 {demo-c}/demo-in-linux/elf/symbol_elf_nm.md 中。

### 使用符号表的地址直接访问数据

在程序中可以直接使用符号表中的 Value 值去访问对应的内存数据。
同样的源文件，每次编译得到的符号表的地址是一样的。
如果只是修改了某个变量的值，没有大规模的修改代码的话，重新编译的时候，变量的地址一般也是不会变得。
可以通过这种方式验证这个结论。

示例代码：address.c。不过，在 ubuntu 22.04 中，这种方式无效。

### 进程虚拟地址空间映射

需要注意的是，在上面的输出中，与地址有关的数据，都不是程序跑起来的时候，在内存中真正的地址。
程序在进程里跑起来的时候，操作系统会把真正的内存地址和 elf 文件中的虚拟地址做映射。
