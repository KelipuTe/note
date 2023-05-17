---
draft: false
create_date: 2021-12-06 08:00:00 +0800
date: 2023-05-16 08:00:00 +0800
title: "在 Linux 系统中使用 C 语言进行编程的注意点"
summary: "学会看 Linux 文档；注意代码运行的目标环境；笔记中出现的文档；"
toc: true

categories:
- operating-system(操作系统)

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- linux-c
---
## 正文

### 学会看 Linux 文档

想在 Linux 系统中使用 C 语言进行编程，一定要学会看 Linux 的文档。第一，不会看文档，那搞个屁的编程。第二，遇到问题，仔细看看文档，一般都能解决。

#### Linux man pages online

在线文档可以从 [man7.org](https://man7.org/index.html) 进去。在 man7.org 页面，点击 Online manual pages，可以进到 [Linux man pages online](https://man7.org/linux/man-pages/index.html) 页面。

在 Linux man pages online 页面，点击 by section 链接，可以进到一个列表页面。在这个列表页面，有所有的条目。虽然，东西多，但是，可以用浏览器进行全文搜索。找起来比较容易，不需要知道要找的条目到底在哪个 page。

#### man 命令

在 Linux 操作系统中，可以使用 man 命令查看 Linux 文档。但是用 man 命令的时候，需要知道要找的条目到底在哪个 page。

比如：页面上的 "execve(2) - execute program" 用 man 命令就是 "man 2 execve"。页面上的 "exec(3) - execute a file" 用 man 命令就是 "man 3 exec"。

#### 在 docker 的 centos 中无法使用 man 命令

因为，docker 的 centos，它把一些东西精简了，所以，有一些命令不能使用。比如，man 命令。

需要修改 "/etc/yum.conf" 配置文件。注释掉 `tsflags=nodocs` 这一行。这个配置禁用了一些软件包。修改配置文件后，重新安装 man 命令，即可使用。

```
> yum -y install man
> yum -y install man-pages
```

### 注意代码运行的目标环境

编程的时候，要注意代码运行的目标环境。同一段代码，在 Ubuntu 和 Centos 上运行的时候，整体的系统调用过程应该是差不多的，但是细节可能不一样。

举个例子，在 Ubuntu 22.04 和 Centos 7 上分别跑 hello world 程序。加载 libc.so.6 文件的这个步骤。在 Ubuntu 22.04 上加载的是 "/lib/x86_64-linux-gnu/libc.so.6"；而在 Centos 7 上加载的是 "/lib64/libc.so.6"。

### 笔记中出现的文档

笔记中出现的 Linux 文档，都会统一写在这里。直接 by section 页面搜就可以。不会散步到每一篇笔记中去，每一个都贴链接，太麻烦了。

笔记中出现的会是从文档中节选的关键的部分，或者和实践过程有关的部分。有可能是全的，有可能是省略的，这不一定。所以，看的时候稍微注意一点。

#### 进程

| 标题                                                        | 描述 |
|-----------------------------------------------------------| --- |
| **进程的标识**                                                 | --- |
| getuid(2) - get user identity                             | --- |
| geteuid(2) - get user identity                            | --- |
| getpid(2) - get process identification                    | --- |
| getppid(2) - get process identification                   | --- |
| getpgid(2) - set/get process group                        | --- |
| **进程的创建**                                                 | --- |
| fork(2) - create a child process                          | --- |
| vfork(2) - create a child process and block parent        | --- |
| **进程的运行**                                                 | --- |
| execve(2) - execute program                               | --- |
| exec(3) - execute a file                                  | --- |
| **进程的退出**                                                 | --- |
| exit(2) - terminate the calling process                   | --- |
| _Exit(2) - terminate the calling process                  | --- |
| _exit(2) - terminate the calling process                  | --- |
| exit(3) - cause normal process termination                | --- |
| exit_group(2) - exit all threads in a process             | --- |
| **进程的回收**                                                 | --- |
| wait(2) - wait for process to change state                | --- |
| waitpid(2) - wait for process to change state             | --- |
| **进程的运行顺序**                                               | --- |
| nice(1) - run a program with modified scheduling priority | --- |
| renice(1) - alter priority of running processes           | --- |
| getpriority(2) - get/set program scheduling priority      | --- |
| setpriority(2) - get/set program scheduling priority      | --- |
| nice(2) - change process priority                         | --- |
| **进程的内存资源**                                               | --- |
| proc(5) - process information pseudo-filesystem           | --- |
| getrlimit(2) - get/set resource limits                    | --- |
| setrlimit(2) - get/set resource limits                    | --- |
| ---                                                       | --- |

#### 信号

| 标题                                                     | 描述 |
|--------------------------------------------------------| --- |
| signal(7) - overview of signals                        | --- |
| sigaction(2) - examine and change a signal action      | --- |
| sigprocmask(2) - examine and change blocked signals    | --- |
| sigpending(2) - examine pending signals                | --- |
| sigsetops(3) - POSIX signal set operations             | --- |
| alarm(2) - set an alarm clock for delivery of a signal | --- |
| kill(2) - send signal to a process | 发送信号给一个过程 |
| ---                                                    | --- |

#### 进程间通信

| 标题                                                     | 描述 |
|--------------------------------------------------------| --- |
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

#### 网络间进程间通信

| 标题                                                     | 描述                  |
|--------------------------------------------------------|---------------------|
| socket(7) - Linux socket interface                     | socket 概述           |
| socket(2) - create an endpoint for communication       | 创建 socket           |
| ip(7) - Linux IPv4 protocol implementation             | 设置 ipv4 的 tcp、udp 等 |
| tcp(7) - TCP protocol                                  | TPC 概述              |
| udp(7) - User Datagram Protocol for IPv4               | UDP 概述              |
| unix(7) - sockets for local interprocess communication | UNIX Socket         |
| bind(2) - bind a name to a socket                      | 绑定地址和端口             |
| listen(2) - listen for connections on a socket         | 监听 socket           |
| accept(2) - accept a connection on a socket            | 接受连接                |
| connect(2) - initiate a connection on a socket         | 发起连接                |
| recv(2) - receive a message from a socket              | 从 socket 读取数据       |
| recvfrom(2) - receive a message from a socket          | 从 socket 读取数据       |
| send(2) - send a message on a socket                   | 向 socket 发送数据       |
| sendto(2) - send a message on a socket                 | 向 socket 发送数据       |
| ---                                                    | --- |

#### 文件

| 标题                                              | 描述 |
|-------------------------------------------------| --- |
| access(2) - check user's permissions for a file | --- |
| open(2) - open and possibly create a file       | --- |
| read(2) - read from a file descriptor           | --- |
| write(2) - write to a file descriptor           | --- |
| fcntl(2) - manipulate file descriptor           | 操作文件描述符 |
| ---                                             | --- |

#### 终端

| 标题                                                                       | 描述 |
|--------------------------------------------------------------------------| --- |
| tty(1) - print the file name of the terminal connected to standard input | --- |
| tty(4) - controlling terminal                                            | --- |
| pty(7) - pseudoterminal interfaces                                       | --- |
| pts(4) - pseudoterminal master and slave                                 | --- |
| ptmx(4) - pseudoterminal master and slave                                | --- |
| posix_openpt(3) - open a pseudoterminal device                           | --- |
| ---                                                                      | --- |

#### 其他

| 标题                                                                                | 描述                 |
|-----------------------------------------------------------------------------------|--------------------|
| man(1) - an interface to the system reference manuals                             | man 命令怎么用          |
| errno(3) - number of last error                                                   | errno 怎么用          |
| htons(3) - convert values between host and network byte order                     | 主机字节序和网络字节序的转换     |
| inet_addr(3) - Internet address manipulation routines                             | 点分十进制和二进制网络字节顺序的转换 |
| strncasecmp(3) - compare two strings ignoring case                                | 比较两个字符串，忽略大小写      |
| ---                                                                               | ---                |
