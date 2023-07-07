---
draft: false
date: 2023-07-06 08:00:00 +0800
title: "GDB 调试工具"
summary: "GDB 调试工具"
toc: true

categories:
  - application(应用)

tags:
  - computer-science(计算机科学)
  - application(应用)
  - gdb
---

## 正文

### 官方文档

官方手册：[Debugging with GDB](https://sourceware.org/gdb/current/onlinedocs/gdb.html/)

所有的命令：[Command, Variable, and Function Index](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Command-and-Variable-Index.html)

### gcc 编译

使用 gcc 编译源代码的时候，使用 `-g` 参数就可以让程序携带调试信息。

```
> gcc -g hello.c -o hello.elf
```

### 进入和退出调试

进入调试的时候有两种情况，程序未运行和程序正在运行。

程序未运行的话，可以用这两种。

- gdb {文件名}
- gdb --silent {文件名}

程序正在运行的话，可以用这两种。

- gdb -p {pid}
- gdb --silent，先进 gdb，然后 attach {pid}

退出的话，如果是调试状态，直接在命令行里用 quit 就可以退出了。但是，如果是生产环境，这样是不行的。
生产环境退出的时候，先在命令行 detach，这样就不会影响被调试的进程，然后 quit。

gdb 也可以调试 go 程序和汇编程序，操作是一样的。

### tab

tab 键可以补全命令，如果有多个的话，会列出全部符合的。

### help

help {command}，输出 command 命令怎么用。

### set、show

set 用于设置参数或者配置，show 用于查看当前的参数或者配置。
这两个命令放到具体的场景中说，就不在这里细说了。

### 启动调试

有两个命令可以启动程序，start 和 run。这两个的区别在于：
start 会执行到程序的第一行代码停住。比如，main() 的第一行代码。
run，简写 r，除非有断点，否则就会一直执行到程序结束。

start 还可以输入命令行参数。比如，start {args1} {args2}。
在 start 或者 run 之前，用 set args 也可以给程序输入命令行参数。比如，set args {args1} {args2}。

### info

info，简写 i，输出信息。info 后面可以跟很多东西。

- info os，输出系统信息。
- info proc，输出当前进程信息。
- info proc {pid}，输出当前进程信息。
- info proc mappings，输出进程内存信息。和 /proc/{pid}/maps 内容一样的。
- info args，输出命令行参数。

### list

list，简写 l，输出代码，默认 10 行。输入一次后，继续按回车可以继续输出。
默认 10 行的这个值可以设置可以通过 set listsize {行数 n|unlimited} 修改为 {n|无限}。

- list {行号 n}，输出从第 n 行开始的代码。
- list {行号 n, 行号 m}，输出从第 n 行到第 m 行的代码。
- list {函数名}，输出函数的代码。

info line，输出当前调试在第几行。

### 执行代码

有两个命令可以执行代码，next 和 step。这两个的区别在于：
next，简写 n，逐行执行，会跳过函数。step，简写 s，逐行执行，会进入函数。

{next|step} {行数 n}，执行 n 行代码。

step 默认只会进入程序员编写的函数，不会进入 printf() 这样的底层函数。
因为 gcc -g 只会给程序员编写的代码添加调试信息，printf() 是不带调试信息的。
需要设置 set step-mode on 才可以进入 printf() 这样的底层函数。

### 调试函数

- info functions，输出所有函数。
- info functions {模糊函数名str}，输出所有含有 str 的函数。
- whatis {函数名}，打印函数的定义。

- where，输出函数的调用栈。
- backtrace，简写 bt，输出函数的调用栈。

输出顺序是从内到外，也就是从现在调试停住的位置，往上追溯一直到 main()。
输出的内容里包括，栈帧编号、内层是从哪个函数的哪一行代码进来的。

info frame，简写 i f，输出栈帧的信息。可以看到寄存器、上一层和下一层的栈帧、函数的参数。

frame {栈帧编号 n}，选择栈帧。
这个命令可以回到栈帧编号 n 对应的那个方法里面。然后，就可以看那个位置的相应的信息。
想回来的话 frame 0 就可以。

{up|down} {栈帧数量 n}，可以往上 n 层移动（往 main() 方向移动），或者往下 n 层移动（往调试停住的位置移动），默认一层。

- call {函数名(变量)}，执行一下函数，打印结果。
- print {函数名(变量)}，执行一下函数，打印结果。

- finish，继续执行直到退出函数。
- return {返回值}，不执行完，直接退出函数。可以设置返回值。

### 断点

- break {行号 n}，简写 b，在第 n 行代码上打一个断点。
- break {*(内存地址)}，在地址上打断点。
- tbreak {行号 n}，打一个临时断点。

info breakpoints，简写 info b 或者 i b，显示断点信息。

输出的内容里，最前面的 Num 是断点编号，Enb 表示断点是否启用，后面还有断点地址和在哪个方法里面。

continue，简写 c，运行到下一个断点。

ignore {断点编号} {次数 n}，忽略断点编号为 1 的断点 n 次。

catch {}，捕获断点。比如，catch syscall write，捕获系统调用 write。

- condition {断点编号} {条件}，条件断点。比如 condition 1 x==5，x 等于 5 的时候停在断点编号为 1 的断点上。
- condition {断点编号}，移除条件

- {disable|enable} {断点编号}，{禁用|启用} 断点。
- delete {断点编号}，删除断点。

- watch {变量名}，添加观察断点，只要变量发生变化，就打印变化前后的值。
- rwatch {变量名}，添加观察断点，只要对变量执行读操作就打印变量的数据。
- awatch {变量名}，添加观察断点，只要对变量执行读写操作就打印变量的数据。

### 打印数据

print {expr}，简写 p，打印表达式的值，可以是变量，也可以是函数。

- print {变量名}，打印变量。
- print *(int *){内存地址}，把内存地址转换成 int * 类型，然后输出内存地址上的值。
- print sizeof(变量)，打印变量的大小。
- print {数组变量名}\[{n}\]@{m}，从数组下标 n 开始，打印 m 个元素。

ptype {变量名}，打印变量的类型。

- set print address on/off，打不打印地址。
- set print array on/off，打印数组的时候是不是竖着打。
- set print array-indexes on/off，打印数组的时候带不带下标。
- set print pretty on/off，打印结构体的时候有没有格式。

============
display 变量名，打印变量，只要变量在当前运行的函数栈里面，就会一直打印
info display，查看处于 display 的变量，会输出变量编号
undisplay {变量编号}，取消变量
============

### 输出内存地址上的数据

x/FMT address，格式化输出内存地址上的数据。

FTM 的格式可以理解成，{输出次数 n}{显示格式 f}{数据长度 s}。

- 输出次数，1 次输出 s 长度的数据，输出 n 次。
- 显示格式：x，16 进制；d，10 进制；t，2 进制；f，浮点数；a，地址；c，字符；s，字符串；
- 数据长度：b，字节；h(halfword)，半字，2 字节；w(word)，字，4 字节；g(giant)，双字，8 字节；

- x/1xb 内存地址，从地址开始，以 16 进制打印 1 个字节。
- x/1xb &(变量名)，从变量的起始地址开始，以 16 进制打印 1 个字节。
- x/s 字符串变量名，打印字符串。

这个格式 print 也可以用。

============

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

### 汇编

disassemble，简写 disas，输出程序的汇编代码。
disassemble {函数名|内存地址}，也可以这么用。

- "-m"，输出汇编和对应的源码
- "-r"，输出汇编和对应的 16 进制机器码

- {show|set} disassembly-flavor {风格}，{查看|设置}输出汇编代码的风格。风格有 att 和 intel。
- {show|set} disassemble-next-line {on|off|auto}，反汇编下一行代码。

si，执行汇编一条指令

- info registers，简写 i r，查看寄存器的值，这里一般显示的是部分的。
- info all-registers，查看所有寄存器的值。

- info registers {寄存器名字}，输出寄存器的值
- p {寄存器名字}，输出寄存器的值

- layout {asm|regs|split|src}，显示汇编调试窗口，{汇编|寄存器+汇编|源码+汇编|源码}。
- tui {reg all|float}，显示{所有的|浮点数}寄存器组。

============
