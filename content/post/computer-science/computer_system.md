---
draft: false
date: 2023-03-07 08:00:00 +0800
lastmod: 2023-03-09 08:00:00 +0800
weight: 110
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

这个部分，主要涉及操作系统和程序。借助 "在 Linux 系统中使用 C 语言进行编程" 搞清楚程序是如何运行的。主要是为了：了解操作系统是什么、了解操作系统的内核和系统调用、了解程序是什么、了解程序的运行过程。

- [基本输入输出系统（Basic Input/Output System、BIOS）](/post/computer-science/operating-system/bios)

程序代码经过编译后最终会变成可执行文件。可执行文件里包括了程序指令和程序数据。可执行文件是操作系统提供的一种对于程序的抽象。操作系统在拿到可执行文件之后就可以把程序指令（指令集里包含的指令）和程序数据拿出来，然后放到硬件上去运行了。

- [在 Linux 系统中使用 C 语言进行编程的注意点](/post/computer-science/operating-system/linux/notice)
- [什么是程序](/post/computer-science/operating-system/linux/program)
- [ELF 文件](/post/computer-science/operating-system/linux/elf)
- [运行 ELF 文件](/post/computer-science/operating-system/linux/exec_elf)
- [进程的创建、进程的运行、进程的内存资源、进程的退出、进程的回收](/post/computer-science/operating-system/linux/process)
- [终端和控制台](/post/computer-science/terminal_console)
- [init 进程、进程间关系、作业、会话、守护进程](/post/computer-science/operating-system/linux/process02)
