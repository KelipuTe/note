---
draft: false
date: 2021-12-06 08:00:00 +0800
lastmod: 2023-02-10 08:00:00 +0800
title: "搞 Linux C 编程的注意点"
summary: "学会看 linux 文档；注意代码运行的目标环境；"
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

### 学会看 linux 文档

搞 linux c 编程，一定要学会看 linux 的文档。遇到问题，仔细看看文档，一般都能解决。

#### man

在 linux 系统中，可以使用 man 命令查看 linux 文档。

关于 man 命令具体怎么用可以看：[man(1) - an interface to the system reference manuals](https://man7.org/linux/man-pages/man1/man.1.html)。

#### Linux man pages online

在线文档从 [man7.org](https://man7.org/index.html) 或者 [Linux man pages online](https://man7.org/linux/man-pages/index.html) 都能进的去。

man7.org 页面点击 Online manual pages 可以进到 Linux man pages online 页面。

Linux man pages online 页面点击 by section 链接可以进到列表页面，在列表页面比较好找。

#### 在 centos 的 docker 容器中无法使用 man 命令

因为 centos 的 docker 的镜像，它把一些东西精简了，所以有一些命令不能使用。比如，man。

需要修改 "/etc/yum.conf" 配置文件。注释掉 `tsflags=nodocs` 这一行。这个配置禁用了一些软件包。修改配置文件后，重新安装 man，即可使用。

```
yum -y install man
yum -y install man-pages
```

### 笔记中出现的文档

错误信息：

- [errno(3) - number of last error](https://man7.org/linux/man-pages/man3/errno.3.html)

进程的标识：

- [getpid(2) - get process identification](https://man7.org/linux/man-pages/man2/getpid.2.html)
- [getppid(2) - get process identification](https://man7.org/linux/man-pages/man2/getppid.2.html)

进程的创建：

- [fork(2) - create a child process](https://man7.org/linux/man-pages/man2/fork.2.html)
- [vfork(2) - create a child process and block parent](https://man7.org/linux/man-pages/man2/vfork.2.html)

进程的运行：

- [execve(2) - execute program](https://man7.org/linux/man-pages/man2/execve.2.html)
- [exec(3) - execute a file](https://man7.org/linux/man-pages/man3/exec.3.html)

进程的运行顺序：

- [nice(1) - run a program with modified scheduling priority](https://man7.org/linux/man-pages/man1/nice.1.html)
- [renice(1) - alter priority of running processes](https://man7.org/linux/man-pages/man1/renice.1.html)
- [getpriority(2) - get/set program scheduling priority](https://man7.org/linux/man-pages/man2/getpriority.2.html)
- [setpriority(2) - get/set program scheduling priority](https://man7.org/linux/man-pages/man2/setpriority.2.html)
- [nice(2) - change process priority](https://man7.org/linux/man-pages/man2/nice.2.html)

进程的内存资源：

- [proc(5) - process information pseudo-filesystem](https://man7.org/linux/man-pages/man5/proc.5.html)
- [getrlimit(2) - get/set resource limits](https://man7.org/linux/man-pages/man2/getrlimit.2.html)
- [setrlimit(2) - get/set resource limits](https://man7.org/linux/man-pages/man2/setrlimit.2.html)

进程的退出：

- [exit(2) - terminate the calling process](https://man7.org/linux/man-pages/man2/exit.2.html)
- [_Exit(2) - terminate the calling process](https://man7.org/linux/man-pages/man2/_Exit.2.html)
- [_exit(2) - terminate the calling process](https://man7.org/linux/man-pages/man2/_exit.2.html)
- [exit(3) - cause normal process termination](https://man7.org/linux/man-pages/man3/exit.3.html)
- [exit_group(2) - exit all threads in a process](https://man7.org/linux/man-pages/man2/exit_group.2.html)

进程的回收：

- [wait(2) - wait for process to change state](https://man7.org/linux/man-pages/man2/wait.2.html)
- [waitpid(2) - wait for process to change state](https://man7.org/linux/man-pages/man2/waitpid.2.html)

### 注意代码运行的目标环境

编程的时候，要注意代码运行的目标环境。同一段代码，在 Ubuntu 和 Centos 上运行的时候，整体的系统调用过程应该是差不多的，但是细节可能不一样。

举个例子，在 Ubuntu 22.04 和 Centos 7 上分别跑 helloworld 程序。加载 libc.so.6 文件的这个步骤。在 Ubuntu 22.04 上加载的是 "/lib/x86_64-linux-gnu/libc.so.6"；而在 Centos 7 上加载的是 "/lib64/libc.so.6"。
