---
draft: false
date: 2023-05-16 08:00:00 +0800
title: "Linux C语言编程"
summary: "实践的环境；学会看 Linux 文档；注意代码运行的目标环境；笔记中出现的文档；"
toc: true

categories:
  - 程序

tags:
  - 计算机
  - 程序
---

## 正文

### 实践的环境

CPU 是 AMD64（x86_64）架构，操作系统是 Windows 11。这个可以通过 `systeminfo` 命令查看。

然后，在 Windows 中，使用 VMware Workstation Pro 16，搭建 Linux 虚拟机，操作系统是 Ubuntu 22.04。

Ubuntu 22.04 的 Linux 版本是 linux version 5.19.0-41-generic。这个可以通过 `cat /proc/version` 命令查看。

然后，在 Linux 中安装 GCC 编译器，版本是 gcc version 11.3.0 (ubuntu 11.3.0-1ubuntu1~22.04)。

### 学会看 Linux 文档

想在 Linux 系统中使用 C 语言进行编程，一定要学会看 Linux 的文档。

笔记中出现的是从文档中节选的和笔记有关联的部分。有可能是整段的，有可能是节选的，看的时候稍微注意一点。

笔记不会贴文档的链接，太麻烦了，只会贴个文档的名字，直接去在线文档的 by section 页面搜就可以。

#### 在线文档

在线文档可以从 [man7.org](https://man7.org/index.html) 进去。
在 man7.org 页面，点击 "Online manual pages"。
可以进到 [Linux man pages online](https://man7.org/linux/man-pages/index.html) 页面。

在 "Linux man pages online" 页面。点击 "by section" 链接，可以进到一个列表页面。
在这个列表页面，有所有的条目。虽然东西多，但是可以用浏览器进行全文搜索，找起来比较容易，不需要知道要找的条目到底在哪个 page。

#### man 命令

在 Linux 操作系统中，可以使用 man 命令查看文档。但是，用 man 命令的时候，需要知道要找的条目到底在哪个 page。

比如，在线文档 by section 页面上的 "execve(2) - execute program"。用 man 命令就是 `man 2 execve`；

比如，在线文档 by section 页面上的 "exec(3) - execute a file"。用 man 命令就是 `man 3 exec`。

#### 在 Docker 的 CentOS 容器中无法使用 man 命令

因为，Docker 的 CentOS 容器，它把一些东西精简了，所以，有一些命令不能使用。

需要修改 "/etc/yum.conf" 配置文件。注释掉 `tsflags=nodocs` 这一行。这个配置禁用了一些软件包。

修改配置文件后，使用命令，重新安装 man 命令，即可使用。`yum -y install man` 和 `yum -y install man-pages`

### 注意代码运行的目标环境

编程的时候，要注意代码运行的目标环境。
同一段代码，在 Ubuntu 和 CentOS 上运行的时候，整体的系统调用过程应该是差不多的，但是细节可能不一样。

举个例子，在 Ubuntu 22.04 和 CentOS 7 上分别跑 hello world 程序。加载 libc.so.6 文件的这个步骤。
在 Ubuntu 22.04 上加载的是 "/lib/x86_64-linux-gnu/libc.so.6"。而在 CentOS 7 上加载的是 "/lib64/libc.so.6"。













#### 命令和工具

| 标题                                                          | 描述       |
|-------------------------------------------------------------|----------|
| nm(1) - list symbols from object files                      | 符号表      |
| objdump(1) - display information from object files          | 文件信息     |
| readelf(1) - display information about ELF files            | elf 信息   |
| strace(1) - trace system calls and signals                  | 跟踪系统调用   |
| man(1) - an interface to the system reference manuals       | 文档       |
| gcc(1) - GNU project C and C++ compiler                     | 编译       |
| file(1) - determine file type                               | 文件类型     |
| size(1) - list section sizes and total size of binary files | 段大小      |
| ldd(1) - print shared object dependencies                   | 文件依赖的动态库 |




#### 进程

| 标题                                                     | 描述      |
|--------------------------------------------------------|---------|
| clone(2) - create a child process                      | 创建子进程   |
| fork(2) - create a child process                       | 创建子进程   |
| vfork(2) - create a child process and block parent     | 创建子进程   |
| wait4(2) - wait for process to change state, BSD style | 回收子进程   |
| execve(2) - execute program                            | 让进程执行程序 |
| exit_group(2) - exit all threads in a process          | 退出进程中所有的线程 |




| **进程的标识**                                                 | ---        |
| getuid(2) - get user identity                             | ---        |
| geteuid(2) - get user identity                            | ---        |
| getpid(2) - get process identification                    | ---        |
| getppid(2) - get process identification                   | ---        |
| getpgid(2) - set/get process group                        | ---        |
| **进程的创建**                                                 | ---        |

| **进程的运行**                                                 | ---        |
| exec(3) - execute a file                                  | ---        |
| **进程的退出**                                                 | ---        |
| exit(2) - terminate the calling process                   | ---        |
| _Exit(2) - terminate the calling process                  | ---        |
| _exit(2) - terminate the calling process                  | ---        |
| exit(3) - cause normal process termination                | ---        |
| **进程的回收**                                                 | ---        |
| wait(2) - wait for process to change state                | ---        |
| waitpid(2) - wait for process to change state             | ---        |
| **进程的运行顺序**                                               | ---        |
| nice(1) - run a program with modified scheduling priority | ---        |
| nice(2) - change process priority                         | ---        |
| renice(1) - alter priority of running processes           | ---        |
| getpriority(2) - get/set program scheduling priority      | ---        |
| setpriority(2) - get/set program scheduling priority      | ---        |
| **进程的内存资源**                                               | ---        |
| proc(5) - process information pseudo-filesystem           | ---        |
| getrlimit(2) - get/set resource limits                    | ---        |
| setrlimit(2) - get/set resource limits                    | ---        |
| ptrace(2) - process trace                                 | 跟踪和控制另一个进程 |

#### 信号

| 标题                                                     | 描述        |
|--------------------------------------------------------|-----------|
| signal(7) - overview of signals                        | ---       |
| sigaction(2) - examine and change a signal action      | ---       |
| sigprocmask(2) - examine and change blocked signals    | ---       |
| sigpending(2) - examine pending signals                | ---       |
| sigsetops(3) - POSIX signal set operations             | ---       |
| alarm(2) - set an alarm clock for delivery of a signal | ---       |
| kill(2) - send signal to a process                     | 发送信号给一个过程 |
| ---                                                    | ---       |

#### 进程间通信

| 标题                                                     | 描述  |
|--------------------------------------------------------|-----|
| **管道**                                                 | --- |
| pipe(7) - overview of pipes and FIFOs                  | --- |
| pipe(2) - create pipe                                  | --- |
| mkfifo(3) - make a FIFO special file (a named pipe)    | --- |
| **System V IPC**                                       | --- |
| ipc(2) - System V IPC system calls                     | --- |
| **System V 消息队列**                                      | --- |
| msgget(2) - get a System V message queue identifier    | --- |
| msgsnd(2) - System V message queue operations          | --- |
| msgrcv(2) - System V message queue operations          | --- |
| msgctl(2) - System V message control operations        | --- |
| **System V 信号量**                                       | --- |
| semget(2) - get a System V semaphore set identifier    | --- |
| semctl(2) - System V semaphore control operations      | --- |
| semop(2) - System V semaphore operations               | --- |
| **System V 共享内存**                                      | --- |
| shmget(2) - allocates a System V shared memory segment | --- |
| shmat(2) - System V shared memory operations           | --- |
| shmdt(2) - System V shared memory operations           | --- |
| **POSIX IPC**                                          | --- |
| mq_overview(7) - overview of POSIX message queues      | --- |
| sem_overview(7) - overview of POSIX semaphores         | --- |
| shm_overview(7) - overview of POSIX shared memory      | --- |
| ---                                                    | --- |

#### 线程

| 标题                                                                 | 描述             |
|--------------------------------------------------------------------|----------------|
| **线程**                                                             | ---            |
| pthreads(7) - POSIX threads                                        | 线程概述           |
| pthread_create(3) - create a new thread                            | 创建线程           |
| pthread_join(3) - join with a terminated thread                    | 子线程合入主线程       |
| pthread_detach(3) - detach a thread                                | 主线程与子线程分离      |
| pthread_exit(3) - terminate calling thread                         | 线程退出           |
| **互斥锁**                                                            | ---            |
| pthread_mutex_init(3p) - destroy and initialize a mutex            | 互斥锁初始化         |
| pthread_mutex_lock(3p) - lock and unlock a mutex                   | 互斥锁加锁          |
| pthread_mutex_unlock(3p) - lock and unlock a mutex                 | 互斥锁释放          |
| **条件变量**                                                           | ---            |
| pthread_cond_init(3p) - destroy and initialize condition variables | 条件变量初始化        |
| pthread_cond_wait(3p) - wait on a condition                        | 等待条件变量         |
| pthread_cond_signal(3p) - signal a condition                       | 唤醒一条被条件变量阻塞的线程 |
| pthread_cond_broadcast(3p) - broadcast or signal a condition       | 唤醒全部被条件变量阻塞的线程 |
| ---                                                                | ---            |


#### 文件

| 标题                                                           | 描述         |
|--------------------------------------------------------------|------------|
| read(2) - read from a file descriptor         | 从文件描述符里读取 |
| write(2) - write to a file descriptor         | 往文件描述符里写入 |
| openat(2) - open and possibly create a file                  | 打开或者创建一个文件 |

| access(2) - check user's permissions for a file              | ---        |
| open(2) - open and possibly create a file                    | ---        |
| fcntl(2) - manipulate file descriptor                        | 操作文件描述符    |
| elf(5) - format of Executable and Linking Format (ELF) files | elf 文件     |
| ---                                                          | ---        |

#### 终端

| 标题                                                                       | 描述  |
|--------------------------------------------------------------------------|-----|
| tty(1) - print the file name of the terminal connected to standard input | --- |
| tty(4) - controlling terminal                                            | --- |
| pty(7) - pseudoterminal interfaces                                       | --- |
| pts(4) - pseudoterminal master and slave                                 | --- |
| ptmx(4) - pseudoterminal master and slave                                | --- |
| posix_openpt(3) - open a pseudoterminal device                           | --- |
| ---                                                                      | --- |

#### 其他

| 标题                                                            | 描述                 |
|---------------------------------------------------------------|--------------------|
| errno(3) - number of last error                               | errno 怎么用          |
| htons(3) - convert values between host and network byte order | 主机字节序和网络字节序的转换     |
| inet_addr(3) - Internet address manipulation routines         | 点分十进制和二进制网络字节顺序的转换 |
| strncasecmp(3) - compare two strings ignoring case            | 比较两个字符串，忽略大小写      |
| vdso(7) - overview of the virtual ELF dynamic shared object   | ---                |
| sscanf(3) - input string format conversion                    | ---                |
| strstr(3) - locate a substring                                | ---                |
| isalpha(3) - character classification functions               | ---                |
| readdir(3) - read a directory                                 | ---                |
| ---                                                           | ---                |
