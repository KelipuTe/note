---
draft: false
date: 2023-12-04 08:00:00 +0800
title: "init 进程"
summary: "init 进程；"
toc: true

categories:
  - 操作系统

tags:
  - 计算机科学
  - 操作系统
  - Linux
---

## 反向链接

[操作系统](/post/computer-science/operating-system/操作系统)；

## 正文

### init 进程

init 程序是 Linux 操作系统中不可缺少的程序之一，
它是一个由内核启动的用户级进程。

内核会在过去曾使用过 init 程序的几个地方查找它，
它的正确位置（对 Linux 系统来说）是 "/sbin/init"。

如果内核找不到 init，它就会试着运行 "/bin/sh"。
如果这两个都运行失败了，操作系统的启动也会失败。

Linux 操作系统的启动首先从 BIOS 开始，接下来进入 boot loader，
由 boot loader 载入内核，进行内核初始化。

内核启动之后（被载入内存，开始运行，初始化所有的设备驱动程序和数据结构之后），
就会通过启动一个用户级程序 init 的方式，完成引导进程。

内核初始化的最后一步就是启动 init 进程，
这个进程是系统的第一个进程，它负责产生其他所有的用户进程。

所以，init 进程始终是第一个进程，其进程编号始终为 1。
其他所有的用户进程每个进程都有父进程，一直上溯的话，
所有的进程会形成一个以 init 进程为根结点的树状结构。

## 正向链接

[进程树](/post/computer-science/operating-system/linux/进程树)；

## 参考

<<init进程>>、<<进程树>>、<<进程组>>、<<作业>>、<<会话>> 这几篇笔记的参考都在这里。

- 百度百科：init进程、Linux Shell、Linux进程管理及作业控制
- [Linux初始化init系统](https://zhuanlan.zhihu.com/p/573503461)
- [Linux-进程、进程组、作业、会话、控制终端详解](https://www.cnblogs.com/JohnABC/p/4079669.html)
- [APUE 2 - 进程组（process group） 会话（session） job](https://www.cnblogs.com/Sven7/p/7442791.html)
- [Linux会话、终端与进程组](https://zhuanlan.zhihu.com/p/563471531)