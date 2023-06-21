---
draft: true
date: 2022-07-04 08:00:00 +0800
lastmod: 2022-07-04 08:00:00 +0800
title: "GDB（GDB 调试工具）"
summary: "GDB（GDB 调试工具）"
toc: true

categories:
- linux

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- gdb
---

> 操作系统架构：x86_64 AMD Ryzen
> 操作系统版本：Ubuntu 2204

> 样例代码 <br/>
> demo-c/demo-in-linux/memory-layout/code_and_data.c

### 编译

使用 `gcc` 编译的时候，选择 `-g` 携带调试信息，然后就可以用 gdb 调试了。

```
> gcc -g code_and_data.c -o code_and_data
```

进入调试。

```
> gdb -silent ./code_and_data
Reading symbols from ./code_and_data...
(gdb)
```

启动调试。

启动调试后，会进入 main 函数，然后停在第一行命令上

```
(gdb) start
Temporary breakpoint 1 at 0x114a: file code_and_data.c, line 9.
Starting program: /mnt/hgfs/demo-c/demo-in-linux/memory-layout/code_and_data
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Temporary breakpoint 1, main () at code_and_data.c:9
9	  int a = 1024;
```

输出进程信息

```
(gdb) info proc
process 2746
cmdline = '/mnt/hgfs/demo-c/demo-in-linux/memory-layout/code_and_data'
cwd = '/mnt/hgfs/demo-c/demo-in-linux/memory-layout'
exe = '/mnt/hgfs/demo-c/demo-in-linux/memory-layout/code_and_data'
```

输出进程内存信息

```
(gdb) info proc mappings
process 2746
Mapped address spaces:

          Start Addr           End Addr       Size     Offset  Perms  objfile
      0x555555554000     0x555555555000     0x1000        0x0  r--p   /mnt/hgfs/demo-c/demo-in-linux/memory-layout/code_and_data
      0x555555555000     0x555555556000     0x1000     0x1000  r-xp   /mnt/hgfs/demo-c/demo-in-linux/memory-layout/code_and_data
      0x555555556000     0x555555557000     0x1000     0x2000  r--p   /mnt/hgfs/demo-c/demo-in-linux/memory-layout/code_and_data
      0x555555557000     0x555555558000     0x1000     0x2000  r--p   /mnt/hgfs/demo-c/demo-in-linux/memory-layout/code_and_data
      0x555555558000     0x555555559000     0x1000     0x3000  rw-p   /mnt/hgfs/demo-c/demo-in-linux/memory-layout/code_and_data
      0x7ffff7d81000     0x7ffff7d84000     0x3000        0x0  rw-p
      0x7ffff7d84000     0x7ffff7dac000    0x28000        0x0  r--p   /usr/lib/x86_64-linux-gnu/libc.so.6
      0x7ffff7dac000     0x7ffff7f41000   0x195000    0x28000  r-xp   /usr/lib/x86_64-linux-gnu/libc.so.6
      0x7ffff7f41000     0x7ffff7f99000    0x58000   0x1bd000  r--p   /usr/lib/x86_64-linux-gnu/libc.so.6
      0x7ffff7f99000     0x7ffff7f9d000     0x4000   0x214000  r--p   /usr/lib/x86_64-linux-gnu/libc.so.6
      0x7ffff7f9d000     0x7ffff7f9f000     0x2000   0x218000  rw-p   /usr/lib/x86_64-linux-gnu/libc.so.6
      0x7ffff7f9f000     0x7ffff7fac000     0xd000        0x0  rw-p
      0x7ffff7fbb000     0x7ffff7fbd000     0x2000        0x0  rw-p
      0x7ffff7fbd000     0x7ffff7fc1000     0x4000        0x0  r--p   [vvar]
      0x7ffff7fc1000     0x7ffff7fc3000     0x2000        0x0  r-xp   [vdso]
      0x7ffff7fc3000     0x7ffff7fc5000     0x2000        0x0  r--p   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
      0x7ffff7fc5000     0x7ffff7fef000    0x2a000     0x2000  r-xp   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
      0x7ffff7fef000     0x7ffff7ffa000     0xb000    0x2c000  r--p   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
      0x7ffff7ffb000     0x7ffff7ffd000     0x2000    0x37000  r--p   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
      0x7ffff7ffd000     0x7ffff7fff000     0x2000    0x39000  rw-p   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
      0x7ffffffde000     0x7ffffffff000    0x21000        0x0  rw-p   [stack]
  0xffffffffff600000 0xffffffffff601000     0x1000        0x0  --xp   [vsyscall]
```

### 查看变量或者函数的地址

(gdb) help p
print, inspect, p
Print value of expression EXP.
Usage: print [[OPTION]... --] [/FMT] [EXP

p 是 print（打印）的缩写，用于打印变量的值或表达式的结果。

`p [expression]` expression 是要打印的变量或表达式

### 查看地址上的值

(gdb) help x

Examine memory: x/FMT ADDRESS.
ADDRESS is an expression for the memory address to examine.
FMT is a repeat count followed by a format letter and a size letter.
Format letters are o(octal), x(hex), d(decimal), u(unsigned decimal),
t(binary), f(float), a(address), i(instruction), c(char), s(string)
and z(hex, zero padded on the left).
Size letters are b(byte), h(halfword), w(word), g(giant, 8 bytes).

x/nfu address n=数据数量 f=显示格式 u=数据长度

"x" 表示以十六进制格式输出
"b"（字节）、"h"（半字，即 2 字节）和 "g"（双字，即 8 字节）