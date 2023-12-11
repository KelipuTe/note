---
draft: false
date: 2023-06-10 08:00:00 +0800
title: "Linux 进程"
summary: "进程是什么；怎么观察进程；"
toc: true

categories:
  - Linux

tags:
  - 计算机科学
  - 操作系统
  - Linux
---

## 反向链接

[进程](/post/computer-science/operating-system/进程)；
[ELF](/post/computer-science/operating-system/linux/ELF)；

## 资料

代码：{demo-c}/demo-in-linux/process/

## 正文

### 进程是什么

可执行文件只是程序而已，程序自己是跑不起来的，需要把程序交给操作系统去执行。

当一个程序被交给操作系统执行时，操作系统首先会创建一个进程。
与此同时，一个线程也会被立刻启动起来去执行程序中的 main 函数。
这个线程就叫主线程，后续创建的线程都是这个主线程的子线程。

线程被称作轻量级进程，它是操作系统调度的最小单元，通常一个进程可以拥有多个线程。

进程不仅局限于程序（那段可执行代码），还包含其他资源。例如，文件、内存、线程、信号等。
进程拥有独立的资源空间，进程中的线程共享进程的资源空间。

简单的理解，程序是静态的，进程是动态的。
程序需要变成进程，才能够真正的运行起来，程序启动起来就变成进程，直到程序运行结束。

### 怎么观察进程

可以通过 strace 命令可以跟踪进程执行时的系统调用和进程接收的信号。
关于 strace 命令具体怎么用，可以看 "strace(1) - trace system calls and signals"。

这里会用到 "-f"、"-i"、"-t"、"-T"、"-p"、"-s"、"-o" 这几个参数。
"-f" 跟踪子进程；"-i" 打印系统调用的地址；"-t" 每一行打印时间；
"-T" 显示系统调用花费的时间；"-p {pid}"指定跟踪的进程号；
"-s {length}" 每一行的长度，默认 32，一般设置 65535；"-o {file name}" 输出到文件；

这里观察一下 hello_world.elf 可执行文件运行的过程。
命令的输出我放在 {demo-c}/demo-in-linux/program/ 目录的 hello_world_objdump.md 文件内。

#### 在 centos 的 docker 容器中使用 strace 命令报错

使用 `strace -p` 命令跟踪进程时报错：

```text
attach: ptrace(PTRACE_SEIZE): Operation not permitted
```

参考官方文档：
[The solution for enabling of ptrace and PTRACE_ATTACH in Docker Containers](https://bitworks.software/en/2017-07-24-docker-ptrace-attach.html)。

启动容器的时候使用 "–-privileged" 参数，让容器内的 root 用户拥有真正的 root 权限。

```shell
docker run -it -p 127.0.0.1:9501:9501 -v {local path}:{docker path} --name={container name} --privileged centos:centos7
```

进入容器，然后使用命令：`echo 0 > /proc/sys/kernel/yama/ptrace_scope`。
将 "/proc/sys/kernel/yama/ptrace_scope" 文件中的值修改成 0。然后就可以使用 `strace -p` 命令跟踪进程了。

## 参考

<<进程树>>、<<进程组>>、<<作业>>、<<会话>> 这几篇笔记的参考都在这里。

- 百度百科：Linux Shell、Linux进程管理及作业控制
- [Linux初始化init系统](https://zhuanlan.zhihu.com/p/573503461)
- [Linux-进程、进程组、作业、会话、控制终端详解](https://www.cnblogs.com/JohnABC/p/4079669.html)
- [APUE 2 - 进程组（process group） 会话（session） job](https://www.cnblogs.com/Sven7/p/7442791.html)
- [Linux会话、终端与进程组](https://zhuanlan.zhihu.com/p/563471531)
- [Linux进程控制](https://www.cnblogs.com/cpsmile/p/4382106.html)
