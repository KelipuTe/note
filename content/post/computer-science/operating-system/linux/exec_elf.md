---
draft: true
create_date: 2021-12-06 08:00:00 +0800
date: 2023-02-07 08:00:00 +0800
title: "运行 ELF 文件"
summary: "进程是什么；可执行文件是怎么运行的；"
toc: true

categories:
- operating-system(操作系统)

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- linux-c
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- VMware Workstation Pro 16
- Ubuntu 22.04
- Linux 5.19.0-32-generic x86_64
- gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0

前置笔记：[ELF 文件](/post/computer-science/operating-system/linux/elf)

## 资料

- [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/process/

## 正文

### 进程是什么

简单的理解，程序启动起来就变成进程，直到程序运行结束。程序是静态的，进程是动态的。进程不仅局限于程序（那段可执行代码），还包含其他资源。例如，文件、内存、线程、信号等。

当一个程序启动时，操作系统就会创建一个进程。与此同时，一个线程也会被立刻启动起来去执行 main 函数，这个线程就叫主线程，后续创建的线程都是这个主线程的子线程。

线程被称作轻量级进程，它是操作系统调度的最小单元，通常一个进程可以拥有多个线程。进程拥有独立的资源空间，线程共享进程的资源空间。

### 怎么观察进程

#### strace

通过 `strace` 命令可以来跟踪进程执行时的系统调用和进程接收的信号。关于 strace 命令具体怎么用可以看：[strace(1) - trace system calls and signals](https://man7.org/linux/man-pages/man1/strace.1.html)。这里会用到 `-f`、`-i`、`-t`、`-T`、`-p`、`-s`、`-o` 几个参数。

大概是：`-f` 跟踪子进程；`-i` 打印系统调用的地址；`-t` 每一行打印时间；`-T` 显示系统调用花费的时间；`-p {pid}` 指定跟踪的进程号；`-s {length}` 每一行的长度，默认 32，一般设置 65535；`-o {file name}` 输出到文件。

#### 在 centos 的 docker 容器中使用 strace 命令报错

使用 `strace -p` 命令跟踪进程时报错：

```
attach: ptrace(PTRACE_SEIZE): Operation not permitted
```

参考官方文档：[The solution for enabling of ptrace and PTRACE_ATTACH in Docker Containers](https://bitworks.software/en/2017-07-24-docker-ptrace-attach.html)。

启动容器的时候使用 `–-privileged` 参数，让容器内的 root 用户拥有真正的 root 权限。

```
docker run -it -p 127.0.0.1:9501:9501 -v {local path}:{docker path} --name={container name} --privileged centos:centos7
```

进入容器，然后使用命令：`echo 0 > /proc/sys/kernel/yama/ptrace_scope`。将 `/proc/sys/kernel/yama/ptrace_scope` 文件中的值修改成0。然后就可以使用 `strace -p` 命令跟踪进程了。

### 观察一下 helloworld 程序运行的过程

可以直接用 `strace -f -i -T -s 65535 ./helloworld` 直接跟踪 `./helloworld` 运行的过程，但是这样观察不到全部的细节。这里用另外一种观察方式，观察输入 `./helloworld` 的那个终端对应的进程。

其实用户的输入，内也是有一个进程来处理的，这个进程会接收并分析用户输入的容，然后做出相应的动作。这里这个进程，会接收到用户输入的 `./helloworld`，然后执行 helloworld 这个可执行文件。

假设有两个终端黑窗口（进程），进程 a 负责进行上面的操作。进程 b 负责观察进程 a。在进程 b 进行跟踪之前，需要现在进程 a 里面，使用 `echo $$` 命令获取当前进程的 pid。然后进程 b 使用 `strace -f -i -T -s 65535 -p {pid} -o strace.log` 命令观察进程 a。

命令输出的结果：

```
strace: Process 5148 attached
strace: Process 5333 attached
^Cstrace: Process 5148 detached
```

终端 a 运行输入 `./helloworld` 执行程序，终端 b 在输出前两行之后，按 ctrl+c 终止监视，会输出第三行。追踪到的内容会输出到 strace.log 里面，下面看一下 strace.log 里面的内容。

#### 最前面的重复出现的 read 和 write

这里截取了 strace.log 里面，最前面的一段。`...` 表示这里省略了代码，全部贴过来太长了。

```
5148  [00007ff14a0d98f4] pselect6(1, [0], NULL, NULL, NULL, {sigmask=[], sigsetsize=8}) = 1 (in [0]) <2.905167>
5148  [00007ff14a0d2992] read(0, ".", 1) = 1 <0.000010>
5148  [00007ff14a0d974d] pselect6(1, [0], NULL, [0], {tv_sec=0, tv_nsec=0}, NULL) = 0 (Timeout) <0.000009>
5148  [00007ff14a0d2a37] write(2, ".", 1) = 1 <0.000014>
5148  [00007ff14a0d98f4] pselect6(1, [0], NULL, NULL, NULL, {sigmask=[], sigsetsize=8}) = 1 (in [0]) <0.104884>
5148  [00007ff14a0d2992] read(0, "/", 1) = 1 <0.000011>
5148  [00007ff14a0d974d] pselect6(1, [0], NULL, [0], {tv_sec=0, tv_nsec=0}, NULL) = 0 (Timeout) <0.000009>
5148  [00007ff14a0d2a37] write(2, "/", 1) = 1 <0.000022>
...
```

这里用上面的第二行 `5148  [00007ff14a0d2992] read(0, ".", 1) = 1 <0.000010>` 做一个说明：`5148` 是进程号；`[00007ff14a0d2992]` 是地址；`read(0, ".", 1)` 是系统调用和调用的时候传入的参数；`= 1` 是前面那个系统调用的返回值；`<0.000010>` 是系统调用消耗的时间。

这段的意思就是，终端 a（进程 5148），是由 `/bin/bash` 程序启动来的（`/bin/bash` 程序就是输入命令的那个终端黑窗口）。这一大段 read 函数和 write 函数，跟踪到的就是用户在终端 a 里用键盘输入 `./helloworld` 的过程。read 是读取用户的输入，write 是将用户的输入显示到屏幕上。

关于 read 函数和 write 函数具体怎么用可以看：[read(2) - read from a file descriptor](https://man7.org/linux/man-pages/man2/read.2.html) 和 [write(2) - write to a file descriptor](https://man7.org/linux/man-pages/man2/write.2.html)。

#### clone

下一个关键的步骤是，调用 clone 函数。

```
...
5148  [00007ff14a0a8bc7] clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, child_tidptr=0x7ff149fbba10) = 5333 <0.000117>
...
```

clone 函数的作用是创建一个进程，这里进程 5148 创建了进程 5333。关于 clone 函数具体怎么用可以看：[clone(2) - create a child process](https://man7.org/linux/man-pages/man2/clone.2.html)。另外，fock 函数也可以创建一个进程，这个后面再说。

#### wait4

下一个关键的步骤是，是调用 wait4 函数。

```
...
5148  [00007ff14a0a845a] wait4(-1,  <unfinished ...>
...
```

父进程调用 wait 函数会进入阻塞状态，直到有子进程退出。关于 wait4 函数具体怎么用可以看：[wait4(2) - wait for process to change state, BSD style](https://man7.org/linux/man-pages/man2/wait4.2.html)。

在这里就是进程 5148 调用 wait4 函数进入阻塞状态，然后操作系统切换到子进程 5333 继续运行。`<unfinished ...>` 出现这个就表示产生了进程切换。

#### execve

下一个关键的步骤是，是调用 execve 函数。

```
...
5333  [00007ff14a0a90fb] execve("./helloworld", ["./helloworld"], 0x564eeda28ec0 /* 55 vars */) = 0 <0.001275>
...
```

进程 5333 调用 execve 函数执行可执行文件 helloworld。关于 execve 函数具体怎么用可以看：[execve(2) - execute program](https://man7.org/linux/man-pages/man2/execve.2.html)。

execve 函数会加载可执行文件的 `.text` （程序指令）和 `.data` （程序数据）到当前进程，并覆盖当前进程。`["./helloworld"]` 是参数值。`0x564eeda28ec0 /* 55 vars */` 是环境参数值。

环境参数是供所有应用程序使用的公共数据。

#### libc.so.6

下一个关键的步骤是，加载 libc.so.6 文件。

```
...
5333  [00007f06cfe3cb38] openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3 <0.000078>
5333  [00007f06cfe3cb88] read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\237\2\0\0\0\0\0@\0\0\0\0\0\0\0\360\300!\0\0\0\0\0\0\0\0\0@\08\0\16\0@\0B\0A\0\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0\20\3\0\0\0\0\0\0\20\3\0\0\0\0\0\0\10\0\0\0\0\0\0\0\3\0\0\0\4\0\0\0000>\36\0\0\0\0\0000>\36\0\0\0\0\0000>\36\0\0\0\0\0\34\0\0\0\0\0\0\0\34\0\0\0\0\0\0\0\20\0\0\0\0\0\0\0\1\0\0\0\4\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\340\177\2\0\0\0\0\0\340\177\2\0\0\0\0\0\0\20\0\0\0\0\0\0\1\0\0\0\5\0\0\0\0\200\2\0\0\0\0\0\0\200\2\0\0\0\0\0\0\200\2\0\0\0\0\0\301D\31\0\0\0\0\0\301D\31\0\0\0\0\0\0\20\0\0\0\0\0\0\1\0\0\0\4\0\0\0\0\320\33\0\0\0\0\0\0\320\33\0\0\0\0\0\0\320\33\0\0\0\0\0\314x\5\0\0\0\0\0\314x\5\0\0\0\0\0\0\20\0\0\0\0\0\0\1\0\0\0\6\0\0\0\360H!\0\0\0\0\0\360X!\0\0\0\0\0\360X!\0\0\0\0\0\230O\0\0\0\0\0\0`%\1\0\0\0\0\0\0\20\0\0\0\0\0\0\2\0\0\0\6\0\0\0\300{!\0\0\0\0\0\300\213!\0\0\0\0\0\300\213!\0\0\0\0\0\320\1\0\0\0\0\0\0\320\1\0\0\0\0\0\0\10\0\0\0\0\0\0\0\4\0\0\0\4\0\0\0P\3\0\0\0\0\0\0P\3\0\0\0\0\0\0P\3\0\0\0\0\0\0000\0\0\0\0\0\0\0000\0\0\0\0\0\0\0\10\0\0\0\0\0\0\0\4\0\0\0\4\0\0\0\200\3\0\0\0\0\0\0\200\3\0\0\0\0\0\0\200\3\0\0\0\0\0\0D\0\0\0\0\0\0\0D\0\0\0\0\0\0\0\4\0\0\0\0\0\0\0\7\0\0\0\4\0\0\0\360H!\0\0\0\0\0\360X!\0\0\0\0\0\360X!\0\0\0\0\0\20\0\0\0\0\0\0\0\220\0\0\0\0\0\0\0\10\0\0\0\0\0\0\0S\345td\4\0\0\0P\3\0\0\0\0\0\0P\3\0\0\0\0\0\0P\3\0\0\0\0\0\0000\0\0\0\0\0\0\0000\0\0\0\0\0\0\0\10\0\0\0\0\0\0\0P\345td\4\0\0\0L>\36\0\0\0\0\0L>\36\0\0\0\0\0L>\36\0\0\0\0\0\314p\0\0\0\0\0\0\314p\0\0\0\0\0\0\4\0\0\0\0\0\0\0Q\345td\6\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\20\0\0\0\0\0\0\0R\345td\4\0\0\0\360H!\0\0\0\0\0\360X!\0\0\0\0\0\360X!\0\0\0\0\0\0207\0\0\0\0\0\0", 832) = 832 <0.000078>
...
```

调用 openat 函数 和 read 函数加载 helloworld 程序所依赖的库文件 libc.so.6。

其中 openat 函数会打开 libc.so.6 文件。关于 openat 函数具体怎么用可以看：[openat(2) - open and possibly create a file](https://man7.org/linux/man-pages/man2/openat.2.html)。

openat 函数的返回值 3 是个文件描述符。文件描述符是指当前进程在访问的文件，这个值一般大于等于 0。linux 进程默认情况下会有 3 个缺省打开的文件描述符，分别是标准输入=0， 标准输出=1， 标准错误=2。然后后面的 read 函数的第一个参数就是这个 3，表示从文件里读数据。

libc.so.6 是共享目标文件，也叫共享库、运行库、动态库。用户程序会调用运行库（CRT，C Runtime Library）。运行库封装了操作系统更底层的系统调用函数。Linux、Windows、Mac，这些系统它们的底层接口都是不一样的，并且比较原始且底层，直接使用比较复杂，所以要封装这些比较底层的系统调用。

#### write

下一个关键的步骤是，是调用 write 函数。

```
...
5333  [00007f06cfcf1a37] write(1, "hello, world\n", 13) = 13 <0.000096>
...
```

调用 write 函数输出 `hello, world\n` 到屏幕上。返回值 13 表示 `hello, world\n` 有 13 个字节。

在源码里面，用的是 printf 函数。这个函数声明在 stdio.h 头文件里面。它的底层实现最终调用的就是 write 函数，而 write 函数的具体实现就在 libc.so.6 库里。write 函数就是暴露出来的最底层的函数了，再往下就是驱动和硬件相关了。

另外，程序是可以直接调用系统调用函数的。

```
#include <unistd.h>

int main() {
  write(1, "hello, world\n", 13);
  return 0;
}
```

#### exit_group

下一个关键的步骤是，是调用 exit_group 函数。关于 exit_group 函数具体怎么用可以看：[exit_group(2) - exit all threads in a process](https://man7.org/linux/man-pages/man2/exit_group.2.html)。

```
...
5333  [00007f06cfcc7ca1] exit_group(0)  = ?
5333  [????????????????] +++ exited with 0 +++
...
```

进程 5333 调用 exit_group(0) 函数退出进程，参数值 0 是进程退出状态码，就是程序里 return 的 0。

#### wait4

下一个关键的步骤是，是进程 5148 从 wait4 函数的阻塞状态中被唤醒。

```
...
5148  [00007ff14a0a845a] <... wait4 resumed>[{WIFEXITED(s) && WEXITSTATUS(s) == 0}], WSTOPPED|WCONTINUED, NULL) = 5333 <0.036896>
...
```

父进程 5148 知道子进程 5333 退出了，前面调用的 wait4 函数会回收退出的子进程 5333，并回收子进程的内存资源。`WIFEXITED(s)` 和 `WEXITSTATUS(s)` 是宏函数，`WEXITSTATUS(s)` 可以拿到退出状态码。

#### 至此整个 helloworld 程序执行结束。

## 参考（reference）

- [百度百科-主线程](https://baike.baidu.com/item/%E4%B8%BB%E7%BA%BF%E7%A8%8B/9600138?fr=aladdin)
