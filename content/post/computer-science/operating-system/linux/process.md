---
draft: false
date: 2021-12-08 08:00:00 +0800
lastmod: 2023-02-15 08:00:00 +0800
title: "进程的创建、运行、退出"
summary: ""
toc: true

categories:
- operating-system(操作系统)

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- linux-c
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> VMware Workstation Pro 16<br/>
> Ubuntu 22.04<br/>
> gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0

### 前言

### 资料

- [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/process/

### 进程创建

进程创建主要涉及 fock()、vfock()。

### fock()

#### fock() 是什么

> DESCRIPTION</br>
> fork() creates a new process by duplicating the calling process.</br>
> The new process is referred to as the child process.</br>
> The calling process is referred to as the parent process.</br>
> ...</br>

fork() 通过复制调用进程创建一个新进程，新进程为子进程，调用进程为父进程。

#### fork() 的返回值

> RETURN VALUE</br>
> On success, the PID of the child process is returned in the parent, and 0 is returned in the child.</br>
> On failure, -1 is returned in the parent, no child process is created, and errno is set to indicate the error.

- 成功时，父进程拿到子进程的 pid，子进程拿到 0。可以根据这个判断哪个是父进程，哪个是子进程。
- 失败时，父进程拿到-1，子进程不会被创建，errno 会被设置用于表示错误。

#### pid 和 ppid

getpid() 返回调用进程的 pid，getppid() 返回调用进程的父进程的 pid。在使用时需要注意，必须让子进程先执行，父进程后执行，打印出来的 ppid 才是正确的。

如果父进程在子进程执行前先跑完了，那么子进程打印出来的 ppid 就会变成 1。因为父进程已经没了，子进程变成了孤儿进程。孤儿进程会被 1 号进程接管，有可能会变成后台进程。

1 号进程就是 init 进程。init 进程是所有其他进程的祖先进程，它是 linux 系统启动过程中的第一个进程，也是系统在运行时的第一个用户级进程。init 进程的 pid（进程标识符）是就是 1。

#### 子进程和父进程的内存空间

> DESCRIPTION</br>
> ...</br>
> The child process and the parent process run in separate memory spaces.</br>
> At the time of fork() both memory spaces have the same content.</br>
> Memory writes, file mappings (mmap(2)), and unmappings (munmap(2)) performed by one of the processes do not affect the other.</br>
> ...

两个进程运行在不同的内存空间，进程间是隔离的。在 fork() 时，两个进程的内存空间的内容是一样的（程序数据和程序指令）。两个进程进行写内存操作（定义新的变量并赋值，修改已定义的变量的值，定义新的函数）或者文件映射（进程间通信）时互不影响。

在 fork() 时，子进程和父进程代码是一样的，子进程会从 fork() 的下一行代码开始继续执行。一般是父进程先被调度，除非父进程被阻塞了。

#### copy on write（写时复制）

示例详见：{demo-c}/demo-in-linux/process/copy_on_write.c

在 fork() 执行之后 exec() 执行之前，两个进程用的是相同的物理空间，子进程的代码段、数据段、堆栈都是指向父进程的物理空间。两者的虚拟空间不同，但其对应的物理空间是同一个。当父子进程中有更改相应段的行为发生时，再为子进程相应的段分配物理空间。

如果没有执行 exec()，内核会给子进程的数据段、堆栈段分配相应的物理空间（两者有各自的进程空间，互不影响）。 而代码段继续共享父进程的物理空间（两者的代码完全相同）。而如果执行了 exec()，由于两者执行的代码不同，子进程的代码段也会被分配单独的物理空间。

#### 子进程和父进程的区别

> DESCRIPTION</br>
> ...</br>
> - The child has its own unique process ID, and this PID does not match the ID of any existing process group (setpgid(2)) or session.</br>
> - The child's parent process ID is the same as the parent's process ID.</br>
> - The child does not inherit its parent's memory locks (mlock(2), mlockall(2)).</br>
> ...

- 子进程有自己独立的唯一的进程标识（pid）。
- 子进程的父进程的 pid（ppid）和父进程的 pid 是一样的。
- 子进程不会继承父进程的内存锁。

### vfork()

#### vfork() 是什么

> Linux description</br>
> vfork(), just like fork(2), creates a child process of the calling process.</br>
> For details and return value and errors, see fork(2).</br>
> ...</br>
> vfork() differs from fork(2) in that the calling thread is suspended until the child terminates (either normally, by calling_exit(2), or abnormally, after delivery of a fatal signal), or it makes a call to execve(2).</br>
> Until that point, the child shares all memory with its parent, including the stack.</br>
> ...

vfork() 和 fork() 用法一样。区别在于，vfork() 创建子进程后，父进程会被阻塞，直到子进程退出。而且 vfork() 创建出来的子进程和父进程共享内存，包括栈。

#### vfork() 有 bug

当代码使用 `return 0` 结束或者执行到最后 1 行代码结束时，有可能会报 `Segmentation fault (core dumped)` 错误。但是使用 `exit(0)` 或者 `_exit(0)` 结束的时候不会。

通过 strace 命令追踪可以发现：报错时，子进程调用 `exit_group(0)` 退出，但是父进程没有调用 `exit_group(0)` ；不报错时，两个进程都调用 `exit_group(0)` 退出。

这里猜测应该是共享内存的问题，如果子进程退出的时候把栈干碎了，那父进程被拉起来的时候，就没有栈，肯定会报错。

### 进程运行

进程运行主要涉及 execve()。

### execve()













在程序里可以使用的和 execve() 函数作用相同的函数有好几个，这里用 execv() 函数举例。代码详见`demo_c/demo_linux_c/execv/execv.c`和`demo_c/demo_linux_c/execv/call.c`。

#### 进程的执行顺序

进程的执行（调度）顺序受到PRI（priority）值和NI（nice）值控制，这两个值越小，进程优先级越高。这两个值可以通过`ps -ely`命令查看（PRI和NI参数），也可以通过`top`命令查看（PR和NI参数）。

可以使用`nice`命令和`renice`命令调整进程的优先级。nice的值的范围是-20~19。`nice`命令用于进程启动之前（`nice - run a program with modified scheduling priority`）。`renice`命令用于进程启动之后（`renice - alter priority of running processes`）。

在代码中，getpriority()函数可以查看进程优先级，setpriority()函数和nice()函数可以调整进程优先级。详见linux文档`getpriority(2)`、`setpriority(2)`、`nice(2)`。

这里需要注意的是，getpriority()函数和setpriority()函数的which参数选什么，who就要对应的填什么。代码详见`demo_c/demo_linux_c/nice/nice.c`。


### 进程退出的方式

- 程序运行到最后一行代码
- 进程调用`exit()`函数退出进程
- 进程调用`exit_group()`函数
- 进程调用`_exit()`函数退出进程
- 进程调用`_Exit()`函数退出进程
- 进程接收到了中断信号

调用`abort()`函数，进程会被异常终止。

### 不同退出方式的区别

程序运行到最后1行代码、代码`return 0`、调用`exit()`函数退出进程时，会输出输出缓冲区内的内容。而调用`_exit()`函数、调用`_Exit()`函数退出进程时不会。也就是对于`printf("hello, world\r\n")`来说，前面3个方式会输出`hello, world`，后面两个不会输出。

### 退出状态码

当进程结束时，会返回1个退出状态码，一般都是0，0表示成功退出。

使用`echo $?`命令可以打印上一个程序的退出状态码。

退出状态码最大1个字节（也就是最大只能是255），大于1个字节会被处理（值和255逻辑与）。比如返回300，最后会被处理成44。

```
[pid   204] [00007f6e21067da9] exit_group(300) = ?
[pid   204] [????????????????] +++ exited with 44 +++
```

子进程退出时会向父进程发送SIGCHLD中断信号，同时父进程调用`wait4()`函数也会返回并且得到退出的相关信息。

```
[00007f67fa7f7630] --- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=204, si_uid=0, si_status=44, si_utime=0, si_stime=0} ---
[00007f67fa88646c] wait4(-1, 0x7ffdc7754a50, WNOHANG|WSTOPPED|WCONTINUED, NULL) = -1 ECHILD (No child processes)
```

### 进程回收

当子进程调用`exit(0)`退出，但是父进程没有运行结束而且没有回收子进程的时候，子进程会变成僵尸进程。当父进程也结束的时候，操作系统会回收僵尸进程和父进程。

当进程变成僵尸进程时，它的内存数据还驻留在内存中，同时在`/proc`目录还有相关文件没有移除掉，还在占用系统资源。如果僵尸进程过多，会导致系统资源紧张，会影响操作系统的运行。所以我们必须要回收退出的子进程。

可以使用`wait()`函数或`waitpid()`函数回收子进程。`wait()`函数和`waitpid()`函数会阻塞父进程，直到有子进程退出。

#### 示例

- `demo_c/demo_linux_c/wait/wait.c`，通过`wait()`函数回收子进程。
- `demo_c/demo_linux_c/waitpid/waitpid_status.c`，通过`waitpid()`函数回收正常退出的子进程。
- `demo_c/demo_linux_c/waitpid/waitpid_signal.c`，通过`waitpid()`函数回收被信号终止的子进程。
- `demo_c/demo_linux_c/waitpid/waitpid_signal_handler.c`，信号处理函数。

程序运行后，在另外一个终端上，通过`kill -s SIGUSR1 {pid}`命令向子进程发送SIGUSR1信号。可以通过`kill -l`命令查看所有的信号。

测试信号处理函数时，当测试的是信号是SIGSTOP时，并不会像预期的那样输出SIGSTOP信号的值。因为进程收到SIGSTOP后已经停止作业（并没有退出）。如果这时再向进程发送SIGCONT信号。这时进程会恢复作业，并输出`signal no=18`，也就是收到的SIGCONT信号的值。

通过strace命令检测，可以得到下面的内容。进程确实收到了SIGSTOP信号和SIGCONT信号。

```
--- SIGSTOP {si_signo=SIGSTOP, si_code=SI_USER, si_pid=84, si_uid=0} ---
--- stopped by SIGSTOP ---
--- SIGCONT {si_signo=SIGCONT, si_code=SI_USER, si_pid=84, si_uid=0} ---
write(1, "signal no=18\r\n", 14)        = 14
```