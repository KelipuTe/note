---
draft: false
date: 2021-12-12 08:00:00 +0800
lastmod: 2022-04-16 08:00:00 +0800
title: "信号"
summary: "硬件中断；软件中断；中断处理例程和中断处理程序；信号对进程的影响；信号的产生；信号的使用；信号屏蔽和未决信号；"
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

## 正文

### 硬件中断

硬件中断（hardware interrupt）是一个由硬件设备产生的信号，它需要 CPU 关注。硬件中断用于允许硬件设备与 CPU 进行通信，而不需要一直轮询或者等待 CPU 关注。

当一个硬件设备产生一个中断时，它向中断控制器发送一个信号，然后中断控制器（interrupt controller）向 CPU 发送一个信号，中断其当前任务并处理该设备的请求。然后，CPU 将停止执行其当前任务并处理中断，这包括暂时保存 CPU 的状态和执行中断处理例程（interrupt handler routine）。

中断处理例程是一段由 CPU 响应中断而执行的代码。它负责确定是哪个设备产生了中断，处理来自该设备的请求，并恢复 CPU 的状态，以便它能够恢复其先前的任务。

硬件中断对于计算机系统的正常运行至关重要，因为它们允许硬件设备与 CPU 进行交互，而不会垄断 CPU 的资源。产生中断的硬件设备的常见例子包括键盘、鼠标、网卡和硬盘。

硬件中断可以大概分为下面几部分：中断请求，中断响应，保护现场，中断处理，恢复现场，中断返回。

比如，CPU 正在跑一个程序。这时，按一下键盘（产生中断请求）。这个中断请求被发给 CPU 之后，CPU 会做出反应（中断响应）。

这时，CPU 会先停下正在跑的程序，把程序现在的状态记下来（保护现场）。然后，过来处理这个中断请求（中断处理）。这个时候 CPU 里跑的是另外一个程序（中断处理例程）。

中断处理例程执行结束之后，需要把状态恢复到刚才停下的那个时候（恢复现场）。这次中断就完事了，从中断处理例程中退出来（中断返回）。然后，就可以继续执行程序了。

### 软件中断

软件中断（software interrupt）也被称为陷阱（trap）或异常（exception），是一个由程序产生的信号，它使 CPU 中断其当前任务并执行一个特定的例程（routine）来处理中断。与由外部硬件设备产生的硬件中断不同，软件中断是由 CPU 上运行的程序产生的。

有两种主要的软件中断类型：同步（synchronous）和异步（asynchronous）。同步中断是由 CPU 响应程序中的指令而产生的，比如，除以 0 或无效的内存访问。异步中断是由外部事件产生的，比如，一个定时器或来自另一个进程的信号。

当一个软件中断产生时，CPU 会保存程序的当前状态，包括指令指针和寄存器值，并跳转到一个特定的例程，称为中断处理程序或异常处理程序。中断处理程序负责处理中断，其中可能涉及错误处理、内存管理或 I/O 操作等任务。

中断处理完毕后，CPU 恢复程序的保存状态，并在被中断的地方恢复执行。软件中断是程序与操作系统进行通信和执行需要特权访问的任务的重要机制，比如，系统调用或中断另一个进程。

### 中断处理例程和中断处理程序的区别

中断处理例程（interrupt handler routine）和中断处理程序（interrupt handler）的关键区别在于，中断处理例程是响应硬件设备的中断信号而执行的代码，而中断处理程序是管理中断信号并调用适当的中断处理例程的代码。

#### 信号对进程的影响

> signal(7)</br>
> Linux supports both POSIX reliable signals (hereinafter "standard signals") and POSIX real-time signals.</br>
> Signal dispositions</br>
> Each signal has a current disposition, which determines how the process behaves when it is delivered the signal.</br>
> The entries in the "Action" column of the table below specify the default disposition for each signal, as follows:</br>
> -- Term:Default action is to terminate the process.</br>
> -- Ign:Default action is to ignore the signal.</br>
> -- Core:Default action is to terminate the process and dump core (see core(5)).</br>
> -- Stop:Default action is to stop the process.</br>
> -- Cont:Default action is to continue the process if it is currently stopped.</br>
> A process can change the disposition of a signal using sigaction(2) or signal(2). (The latter is less portable when establishing a signal handler; see signal(2) for details.)  Using these system calls, a process can elect one of the following behaviors to occur on delivery of the signal: perform the default action; ignore the signal; or catch the signal with a signal handler, a programmer-defined function that is automatically invoked when the signal is delivered.

每个信号都有一个当前处置，它决定了进程在收到信号时如何进程在收到信号时的行为。下面是每种信号的默认处理方式：Term：进程终止；Ign：进程忽略；Core：进程终止并产生 "core dump" 文件；Stop：进程停止；Cont：进程继续执行

进程可以更改信号的设置，信号发生时可以选择下面三种行为。执行默认动作（default action，SIG_DFL）；忽略信号（ignore，SIG_IGN）；使用信号处理程序捕捉信号（signal hanlder）

#### 信号的产生

- 终端按下 Ctrl+C，产生 SIGINT 信号；终端按下 Ctrl+\ 产生 SIGQUIT 信号；终端按下 Ctrl+Z 产生 SIGSTOP 信号。
- 进程访问一些不存在的内存或是非法内存，会产生 SIGSEGV 中断信号
- 在终端中，使用 kill 命令发送
- 在进程中，使用 raise()、kill()、alarm() 等发送中断信号
- 子进程退出时会产生中断信号

#### Linux 中的信号

在 Linux 中，中断信号有64个，分位标准信号和实时信号。可以通过 "kill -l" 命令可以查看 Linux 中的 64 个中断信号。其中，带 RT（real time）的就是实时信号。

```
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

### 信号的使用

进程可以更改信号的设置。进程在收到信号后，如果编写了信号处理函数就执行信号处理函数，如果没有编写信号处理函数就会执行默认动作。

代码示例：{demo-c}/demo-in-linux/signal/signal.c。

程序运行起来之后，我们通过 "kill -s" 命令向进程发送信号。

- 如果向程序发送 SIGUSR1，进程会执行默认动作，输出 "User defined signal 1"，然后停止。
- 如果向程序发送 SIGUSR2，进程会忽略信号，什么反应都没有。
- 如果向程序发送 SIGINT，进程会执行处理函数，输出 "\[info\]:signal_no=2"，然后继续执行。

### 信号和系统调用

> signal(7)</br>
> Interruption of system calls and library functions by signal handlers If a signal handler is invoked while a system call or library function call is blocked, then either:</br>
> -- the call is automatically restarted after the signal handler returns; or</br>
> -- the call fails with the error EINTR.
> ...</br>
> If a blocked call to one of the following interfaces is interrupted by a signal handler, then the call is automatically restarted after the signal handler returns if the SA_RESTART flag was used; otherwise the call fails with the error EINTR:</br>

如果进程正在执行一些系统调用，此时进程收到了一个信号，则该系统调用将会停止并返回错误，错误码（errno）一般是 -1，错误（error）一般是 EINTR。

如果进程正在执行系统调用，而且这个系统调用被阻塞。这时来了一个信号，结果会有两种。第一种，进程的信号处理函数已经执行返回，但是系统调用返回错误，错误码为 EINTR。第二种，进程的信号处理函数已经执行返回，这时系统调用会重新开始。

#### 系统调用返回错误

代码示例：{demo-c}/demo-in-linux/signal/sigaction_default.c。

```
[debug]:getpid()=4475
[info]:signal_no=2
[debug]:byte=-1,errno=4,error=Interrupted system call
```

这是终端的输出，我们可以看到，先执行了处理函数，然后 read() 系统调用输出了错误，然后进程就终止了。我们用 strace 再观察一下。

```
[00007febcad14992] read(0, 
```

这是没有发送信号的时候。进程阻塞在 read() 系统调用这里。

```
[00007febcad14992] read(0, 0x7ffd2b0b9ed0, 128) = ? ERESTARTSYS (To be restarted if SA_RESTART is set) <5.907470>
[00007febcad14992] --- SIGINT {si_signo=SIGINT, si_code=SI_USER, si_pid=2304, si_uid=1000} ---
[00007febcad14a37] write(1, "[info]:signal_no=2\n", 19) = 19 <0.000019>
[00007febcac42529] rt_sigreturn({mask=[]}) = -1 EINTR (Interrupted system call) <0.000005>
[00007febcad14a37] write(1, "[debug]:byte=-1,errno=4,error=Interrupted system call\n", 54) = 54 <0.000006>
[00007febcaceaca1] exit_group(0)        = ?
[????????????????] +++ exited with 0 +++
```

这是发送信号之后。进程收到信号之后，先执行了处理函数。处理函数返回时，是 EINTR 错误。然后 read() 系统调用拿到了这个 EINTR 错误。

#### 系统调用重新开始

代码示例：{demo-c}/demo-in-linux/signal/sigaction_restart.c。

```
[debug]:getpid()=4491
[info]:signal_no=2
```

这是终端的输出，可以看到，先执行了处理函数，然后 read() 系统调用并没有和上面一样输出错误，而是继续执行。我们用 strace 再观察一下。

```
[00007f08c8714992] read(0, 
```

这是没有发送信号的时候。进程阻塞在 read() 系统调用这里。

```
[00007f08c8714992] read(0, 0x7ffcdcb49bd0, 128) = ? ERESTARTSYS (To be restarted if SA_RESTART is set) <4.457116>
[00007f08c8714992] --- SIGINT {si_signo=SIGINT, si_code=SI_USER, si_pid=2304, si_uid=1000} ---
[00007f08c8714a37] write(1, "[info]:signal_no=2\n", 19) = 19 <0.000077>
[00007f08c8642529] rt_sigreturn({mask=[]}) = 0 <0.000074>
[00007f08c8714992] read(0,
```

这是发送信号之后。进程收到信号之后，先执行了处理函数。处理函数返回时，没有错误。然后 read() 系统调用重新执行了。

### 信号屏蔽和未决信号

> signal(7)</br>
> A signal may be blocked, which means that it will not be delivered until it is later unblocked.  Between the time when it is generated and when it is delivered a signal is said to be pending.</br>

信号可以被阻塞，这时信号不会交付（执行默认动作或者被交给信号处理函数），直到它不被阻塞。信号处于生成和交付之间的状态，被称为未决。

代码示例：{demo-c}/demo-in-linux/signal/signal_block.c。

程序大体的逻辑是，在 10 秒之内阻塞 SIGINT 信号，第 10 秒的时候解开 SIGINT 信号的阻塞。然后，分别在 10 秒之内和 10 秒以后向进程发送 SIGINT 信号。

```
[debug]:getpid()=4774
[info]:pendingSet:0000000000000000000000000000000
[info]:pendingSet:0000000000000000000000000000000
[info]:pendingSet:0000000000000000000000000000000
[info]:pendingSet:0000000000000000000000000000000
[info]:pendingSet:0100000000000000000000000000000
[info]:pendingSet:0100000000000000000000000000000
[info]:pendingSet:0100000000000000000000000000000
[info]:pendingSet:0100000000000000000000000000000
[info]:pendingSet:0100000000000000000000000000000
[info]:pendingSet:0100000000000000000000000000000
[info]:signal_no=2
[info]:pendingSet:0000000000000000000000000000000
[info]:pendingSet:0000000000000000000000000000000
[info]:signal_no=2
[info]:pendingSet:0000000000000000000000000000000
[info]:pendingSet:0000000000000000000000000000000
```

可以看到，在 10 秒之内和向进程发送 SIGINT 信号的时候，信号处理函数并没有输出。未决信号里面从没有数据变成有数据，说明 SIGINT 信号被阻塞了。

第 10 秒的时候，因为 SIGINT 信号的阻塞被解开，所以，信号处理函数开始工作了。处理完之后，未决信号里面就变成空的了。

10 秒以后向进程发送 SIGINT 信号的时候，信号不会被阻塞，信号处理函数直接开始工作，未决信号里面不会有数据。

### 其他信号

#### SIGKILL、SIGSTOP

> signal(7)</br>
> The signals SIGKILL and SIGSTOP cannot be caught, blocked, or ignored.</br>

SIGKILL 和 SIGSTOP 这两个信号不能被捕捉、阻止、忽略。

#### SIGALRM

用 alarm() 定时触发 SIGALRM 信号，可以实现类似定时任务的结构。

### 观察进程数据

> signal(7)</br>
> The /proc/\[pid\]/task/\[tid\]/status file contains various fields that show the signals that a thread is blocking (SigBlk), catching (SigCgt), or ignoring (SigIgn). (The set of signals that are caught or ignored will be the same across all threads in a process.) Other fields show the set of pending signals that are directed to the thread (SigPnd) as well as the set of pending signals that are directed to the process as a whole (ShdPnd). The corresponding fields in /proc/\[pid\]/status show the information for the main thread.</br>

和信号有关系的进程数据，可以通过 "/proc/{pid}/status" 文件观察。SigQ：当前进程待处理信号数。SigPnd；线程的未决信号。ShnPnd；线程组的未决信号。SigBlk；阻塞的信号。SigIgn；忽略的信号。SigCgt；捕捉的信号。

这里观察一下上面的 {demo-c}/demo-in-linux/signal/signal_block.c 的运行情况。为了观察方便可以拉长 signal_block.c 中的等待时间，方便进行操作。

刚开始运行的时候是这样的。

```
SigQ:	0/15243
SigPnd:	0000000000000000
ShdPnd:	0000000000000000
SigBlk:	0000000000000002
SigIgn:	0000000000000000
SigCgt:	0000000000000002
```

向进程发送 SIGINT 信号之后，变成了这样。

```
SigQ:	1/15243
SigPnd:	0000000000000000
ShdPnd:	0000000000000002
SigBlk:	0000000000000002
SigIgn:	0000000000000000
SigCgt:	0000000000000002
```

解开 SIGINT 信号的阻塞之后，变成了这样。

```
SigQ:	0/15243
SigPnd:	0000000000000000
ShdPnd:	0000000000000000
SigBlk:	0000000000000000
SigIgn:	0000000000000000
SigCgt:	0000000000000002
```

## 参考

- {51CTO学堂}/{可用行师}/[Linux C核心技术](https://edu.51cto.com/course/28903.html)
  - 进程部分、信号部分
- ChatGPT + DeepL
