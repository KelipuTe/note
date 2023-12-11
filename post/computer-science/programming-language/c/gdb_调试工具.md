---
draft: true
date: 2023-07-05 08:00:00 +0800
title: "GDB 调试工具"
summary: "GDB 调试工具"
toc: true

categories:
  - linux

tags:
  - computer-science(计算机科学)
  - programming-language(操作系统)
  - c-programming-language
  - gdb
---

前置笔记：[分库分表](/post/computer-science/database/orm/分库分表)

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

调试没有启动的程序
gdb {文件名}
gdb -q {文件名}
gdb -silent {文件名}

用 quit 就可以退出了

调试正在运行的程序
gdb -silent -pid {pid}
或者 gdb -silent 先进 gdb，然后用 attach {pid} 也可以
退出的时候先用 detach，这样不影响被调试的进程，然后 quit



gdb 也可以调试 go 程序，go 文件编译好之后就可以用 gdb 调试了

gdb 也可以调试汇编，操作是一样的，编译的时候添加 -g 参数

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

### 常用命令

tab 键可以补全命令，如果有多个的话，会列出全部符合的

list，（简写 l），输出代码，默认 10 行，这个值可以设置，输入一次后按回车可以继续输出

list {行号n} 输出从第n行开始的代码
list {行号n,行号m} 输出从第n行到第m行的代码
list {函数名} 输出函数的代码

start，启动调试，执行到第一行代码然后停住，比如main函数的第一行代码
start 还可以给程序输入命令行参数，

在使用 start 或者 run 之前，用 set args 也可以给程序输入命令行参数

run，（简写 r），启动调试，如果没有断点的话，就会直接运行到结束

next，简写 n，逐行执行，会跳过函数
step，简写 s，逐行执行，会进入函数，需要设置 set step-mode on 才可以进入比如 printf 这样不带调试信息的底层函数
step {数字 n}，执行 n 行

where 输出函数的调用栈
会显示是哪个函数的哪行调用的下一个函数，直到现在调试停住的位置，输出顺序是倒序，
一般就是，输出的最上面是调试停住的位置，输出的最下面是main函数在最下面
backtrace，简写 bt，也可以

frame，选择栈帧
frame {数字 n}，结合 where 使用，where 的输出每一行的最前面会有一个数字，就是那个，
这个命令可以回到那个方法里面，然后可以看那个位置相应的信息，想回来的话 frame 0 就可以

info frame，简写 i f，输出栈帧的信息，可以看到寄存器，函数的参数，上下一层的栈帧

info line 当前调试在第几行
info functions 输出所有的函数

up {数字 n} 和 down {数字 n}，可以往上n层，或者往下n层移动，默认一层

call {函数名(变量)}，执行一下函数，打印结果
print {函数名(变量)}，也可以

whatis {函数名}，打印函数的定义

return，退出函数，return {返回值}，不执行完，直接退出函数，并且设置返回值
finish，退出函数

print，（简写 p），打印数据
print {变量名}，打印某个变量
print (int *)内存地址，把某个内存地址转换成 int * 类型，然后打印上面的值

quit 退出 gdb

help 帮助命令，比如 help start

### 断点调试

info breakpoints，（简写，info b，i b），显示断点信息
最前面的Num是断点编号，Enb 表示断点是否启用，后面还有断点地址和在哪个方法里面

break {行号n}，（简写 b），打断点
b {*地址}，在地址上打断点

disable，enable {断点编号} 禁用或者启用断点

continue，简写，c，运行到下一个断点

watch {变量名}，观察断点，可以观察变量的数据
delete {断点编号}，删除断点
rwatch {变量名}，观察断点，只要程序对变量执行读操作就打印变量的数据
awatch {变量名}，观察断点，只要程序对变量执行读写操作就打印变量的数据

condition {断点编号} {条件}，条件断点，比如 condition 1 x==5，x等于5的时候停在断点1上
condition {断点编号}，移除条件

ignore {断点编号} {次数n}，忽略断点n次

catch 捕获断点，catch syscall write，捕获系统调用 write

tbreak {行号n}，打临时断点

### 打印和显示数据

info proc 输出进程的信息
info os {cpus/...}
info line
info args
info files
info maps

set print array on/off 数组按格式打印
set print array-indexes on/off 打印数组的时候带不带下标
set print address on/off 打不打印地址

print x\[0\]@4 ,x是个数组，从下标0打印4个元素

set print pretty on/off 结构体按格式打印

p /x 十六进制打印
p /x 十六进制打印
p /d 十进制打印

x86-64架构下，这东西和芯片架构有关系
half word 半字 2字节
2字节 字 word
4字节 双字 double word
8字节 四字 quad word
16字节 双四字 double quad word

x/1xb 地址，从地址开始以16进制打印1个字节
x/2xb 地址，从地址开始以16进制打印2个字节
x/1db 地址，从地址开始以10进制打印1个字节
x/1tb 地址，从地址开始以2进制打印1个字节

x/s 字符串变量名，地址，打印字符串

p sizeof(变量)，打印变量的大小
ptype 变量，打印变量的类型

p \*(int\*)地址，把地址转换成 int* 再对指针解引用


display 变量名，打印变量，只要变量在当前运行的函数栈里面，就会一直打印
info display，查看处于 display 的变量，会输出变量编号
undisplay {变量编号}，取消变量

### 操作内存

这玩意用的比较少

mem 低地址 高地址 权限
info mem 查看设置的权限，会显示编号
delete mem 编号，删除编号为 n 的 mem

假设有一个 int 变量，地址范围是低地址到高地址
然后 mem 低地址 高地址 ro，设置这块内存只读。rw 就是读写
然后在用 set var \*(int\*)低地址=100 试图修改变量的值的时候，就会报错说不能修改

### 调试多进程

info inferiors 显示进程信息，还有编号，目前调试的进程，在编号前面会有个 *
inferior 编号，切换进程

set follow-fork-mode parent/child 设置 fork 执行的时候跟踪父进程还是子进程，默认parent

这里需要注意的是，父子进程是同时执行的，调试父进程或者子进程的时候子进程或者父进程也在跑

set detach-on-fork on/off 调试一个进程的时候，其他的进程运不运行，默认是运行的

show/set schedule-multiple 这个也可以控制多进程是并发还是不并发

detach inferior 编号，分离进程
kill inferior 编号，kill 进程

### 多线程

info threads 显示线程信息，还有编号，目前调试的线程，在编号前面会有个 *
thread 编号，切换线程

show/set schedule-locking on/off/step 调试一个线程的时候，其他的线程运不运行，默认是step暂停的，但是有特殊情况，这个可以看文档

thread apply all bt full 显示所有线程的局部变量

thread apply 线程编号 命令，可以让命令单独作用于某个线程

### 查看变量或者函数的地址

```text
(gdb) help p
print, inspect, p
Print value of expression EXP.
Usage: print [[OPTION]... --] [/FMT] [EXP
```

p 是 print（打印）的缩写，用于打印变量的值或表达式的结果。

`p [expression]` expression 是要打印的变量或表达式

### 查看地址上的值

```text
(gdb) help x

Examine memory: x/FMT ADDRESS.
ADDRESS is an expression for the memory address to examine.
FMT is a repeat count followed by a format letter and a size letter.
Format letters are o(octal), x(hex), d(decimal), u(unsigned decimal),
t(binary), f(float), a(address), i(instruction), c(char), s(string)
and z(hex, zero padded on the left).
Size letters are b(byte), h(halfword), w(word), g(giant, 8 bytes).
```

x/nfu address n=数据数量 f=显示格式 u=数据长度

"x" 表示以十六进制格式输出
"b"（字节）、"h"（半字，即 2 字节）和 "g"（双字，即 8 字节）

## 参考

- `gdb help`
