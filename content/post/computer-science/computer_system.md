---
draft: false
date: 2023-03-07 08:00:00 +0800
lastmod: 2023-03-07 08:00:00 +0800
title: "计算机系统系列笔记的导航"
summary: "计算机系统系列笔记的导航"
toc: true

categories:
- computer-science(计算机科学)

tags:
- computer-science(计算机科学)
---
## 正文

本篇是计算机系统系列笔记的导航。计算机系统系列笔记是一个持续更新的板块。

### 硬件系统

这个部分，从二极管开始一直到 CPU。主要是为了：了解 CPU 的原理、了解指令集是什么。

指令集是计算机的灵魂，整个计算机都是围绕指令集构建起来的。它一方面指导硬件系统（比如CPU）怎么设计，另一方面指导 BIOS 怎么设计。

- [算术逻辑单元（Arithmetic and Logic Unit、ALU）](/post/computer-science/hardware/alu)
- [内存（Memory）](/post/computer-science/hardware/memory)
- [中央处理器（Central Processing Unit、CPU）](/post/computer-science/hardware/cpu)

### 软件系统

这个部分，主要涉及操作系统和程序。借助"在 Linux 系统中使用 C 语言进行编程"搞清楚程序是如何运行的。主要是为了：了解操作系统是什么、了解操作系统的内核和系统调用、了解程序是什么、了解程序的运行过程。

程序代码经过编译后最终会变成可执行文件。可执行文件里包括了程序指令和程序数据。可执行文件是操作系统提供的一种对于程序的抽象。操作系统在拿到可执行文件之后就可以把程序指令（指令集里包含的指令）和程序数据拿出来，然后放到硬件上去运行了。

- [在 Linux 系统中使用 C 语言进行编程的注意点](/post/computer-science/operating-system/linux/notice)
- [什么是程序](/post/computer-science/operating-system/linux/program)
- [ELF 文件](/post/computer-science/operating-system/linux/elf)
- [运行 ELF 文件](/post/computer-science/operating-system/linux/exec_elf)
- [进程的创建、进程的运行、进程的内存资源、进程的退出、进程的回收](/post/computer-science/operating-system/linux/process)
- [终端和控制台](/post/computer-science/terminal_console)
- [init 进程、进程间关系、作业、会话、守护进程](/post/computer-science/operating-system/linux/process02)

### 本人的一家之言，看不看随意

计算机系统是一个完整的体系，可分为硬件系统和软件系统。软件系统里面还可以分为下层的操作系统和上层的应用程序。应用程序里面还可以分：最下面的各种编程语言；中间层的各种中间件和支持服务；最上层的业务程序。

大多数程序员的日常工作其实是局限于软件系统上层的应用程序的最上层的业务程序的部分。这个部分的特点是内容很多，各种眼花缭乱的有意思的新技术层出不穷。但是再多内容，再新的技术也是需要下层提供支持的。因为最终的程序是要跑在计算机上的，它无法脱离计算机系统。

话说回来，人的精力有限，以有涯逐无涯，殆已。虽然不能掌握全部的细节，但是大概了解每个部分在干什么，还是可以的。要不然，应用层写出来的程序再花哨它也是无源之水，无本之木，它只是恰好跑起来了而已。
