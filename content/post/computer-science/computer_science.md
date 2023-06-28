---
draft: false
date: 2023-06-27 08:00:00 +0800
weight: 110
title: "计算机科学相关笔记的导航"
summary: "计算机科学相关笔记的导航"
toc: true

categories:
  - computer-science(计算机科学)

tags:
  - computer-science(计算机科学)
---

## 正文

本篇是计算机科学相关笔记的导航。

如果某篇笔记有前置笔记，会统一写在前言里面，如果某篇笔记有后置笔记，会直接写在正文里面。

### 硬件系统

这个部分，主要涉及硬件系统的几个基本的硬件单元。
主要是为了，了解 CPU 的基本原理，了解指令集是什么。

指令集是计算机的灵魂，整个计算机都是围绕指令集构建起来的。
它一方面指导硬件系统（比如 CPU）怎么设计，另一方面指导软件系统（比如 BIOS）怎么设计。
指令集就是一个抽象层，是连接硬件系统和软件系统的桥梁。

- [算术逻辑单元（Arithmetic and Logic Unit、ALU）](/post/computer-science/hardware/alu)
- [内存（Memory）](/post/computer-science/hardware/memory)
- [中央处理器（Central Processing Unit、CPU）](/post/computer-science/hardware/cpu)

### 软件系统

这个部分，主要涉及操作系统、程序、在 linux 系统中使用 c 语言进行编程。
了解操作系统是什么、了解操作系统的系统调用、了解程序是什么、了解程序的基本运行过程。

- [基本输入输出系统（Basic Input/Output System、BIOS）](/post/computer-science/operating-system/bios)

- [程序](/post/computer-science/operating-system/program)
- [在 Linux 中使用 C 语言进行编程的注意点](/post/computer-science/operating-system/linux/notice)
- [ELF 文件](/post/computer-science/operating-system/linux/elf)
- [运行 ELF 文件](/post/computer-science/operating-system/linux/exec_elf)
- [系统调用](/post/computer-science/operating-system/system_call)
- [命令行参数和环境参数](/post/computer-science/operating-system/cmd_env_param)
- [动态链接和静态链接](/post/computer-science/operating-system/dynamically_statically_linked)
- [进程的创建、进程的运行、进程的内存资源、进程的退出、进程的回收](/post/computer-science/operating-system/linux/process)
- [终端和控制台](/post/computer-science/terminal_console)
- [init 进程、进程间关系、作业、会话、守护进程](/post/computer-science/operating-system/linux/process02)
- [信号](/post/computer-science/operating-system/linux/signal)
- [线程](/post/computer-science/operating-system/linux/thread)
- [网络间进程间通信](/post/computer-science/network/socket)
- [I/O 模型](/post/computer-science/network/io-model)

## 参考

频繁出现的外部引用不会每一篇笔记里都写，太烦了。
如果是某一篇或者某几篇笔记里引用的，就写在最后的参考里面。

- [ChatGPT](https://chat.openai.com/)
- [Bito](https://bito.ai/)
- [DeepL](https://www.deepl.com/translator)
- {51CTO学堂}/{可用行师}/[Linux C核心技术](https://edu.51cto.com/course/28903.html)
- {51CTO学堂}/{可用行师}/[Golang核心高级](https://edu.51cto.com/course/29852.html)
- {51CTO学堂}/{可用行师}/[内存与数据精讲](https://edu.51cto.com/course/29937.html)
- {51CTO学堂}/{可用行师}/[内存与数据二](https://edu.51cto.com/course/30487.html)