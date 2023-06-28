---
draft: false
date: 2022-01-24 08:00:00 +0800
title: "系统调用"
summary: "什么是系统调用；linux 中的系统调用；"
toc: true

categories:
  - operating-system(操作系统)

tags:
  - computer-science(计算机科学)
  - operating-system(操作系统)
  - linux
  - c-programming-language
---

## 前言

前置笔记：[运行 ELF 文件](/post/computer-science/operating-system/linux/exec_elf)

实践的环境：同 [运行 ELF 文件]()

## 正文

### 什么是系统调用

对于应用软件的开发人员来说，一般都是基于操作系统进行开发的。
一般来说，开发人员开发的应用程序是不能直接操作操作系统的，使用的是操作系统提供的系统接口。
开发人员通过操作系统提供的系统接口实现应用程序的功能。

系统调用为用户级程序访问特权服务和资源提供了一种受控的安全方式，同时保持了操作系统的完整性和安全性。
它们在用户模式和内核模式的执行之间建立了一个界限，使用户程序能够以一种受控的方式与底层操作系统进行交互。

用户进程通过系统调用才能去调用系统资源，当调用系统调用时，用户空间运行的程序会切换到内核空间中去运行。
当启动一个程序之后，程序必须借助操作系统提供的系统接口（系统调用、syscall）才能使用系统的相关资源（一般是硬件资源）。
通常需要系统调用的任务包括文件操作、网络通信、进程管理、内存分配和设备输入输出。

进行系统调用的过程包括以下几个步骤：

- 用户级程序在程序在用户模式下执行其代码，该模式下的权限受到限制，对系统资源的访问也受到限制。
- 当程序需要执行一个特权操作或访问一个系统资源时，就会触发一个系统调用。这通常是通过调用一个特定的系统调用来实现的。
- 系统调用触发了从用户模式到内核模式的过渡。在内核模式下，程序获得了对操作系统的全部权限和资源的访问权。
- 操作系统内核收到系统调用请求并验证其有效性。然后，内核会执行程序所要求的特定操作或服务。
- 一旦系统调用完成，操作系统将控制权返回给用户级程序，过渡到用户模式。
- 用户级程序从一个指定的位置（比如：特定的寄存器或内存位置），检索系统调用的结果或状态，然后，继续其执行自己的代码。

### linux 中的系统调用

系统调用是在 linux 内核的源代码中定义的，而不是专门针对 Ubuntu 的。
linux 内核的源代码，包括系统调用的定义，可以在 "/usr/src/linux" 目录下找到。

c 库的系统调用通常位于 "/usr/include/unistd.h" 文件中。
这个头文件包含了系统调用的函数原型和定义。它会根据系统的架构，加载不同的头文件。

汇编语言、高级语言（c/c++、java、golang）、脚本语言（php、python、nodejs）在运行的时候最终都是调用的系统调用。
这些系统调用看上去是 c 语言风格的定义，但是，程序最终变成汇编代码、机器码的时候是一样的。
这点反过来想比较好理解，芯片是固定的，不可能应付各式各样的编程语言，所以，只能是各式各样的编程语言最终都变成一样的机器语言。

下面是从头文件中节选的一部分部分，就是 read() 和 write() 的定义，和在线文档上是一样的。

```c
/* Read NBYTES into BUF from FD.  Return the
number read, -1 for errors or 0 for EOF.

This function is a cancellation point and therefore not marked with
__THROW.  */
extern ssize_t read (int __fd, void *__buf, size_t __nbytes) __wur
__fortified_attr_access (__write_only__, 2, 3);

/* Write N bytes of BUF to FD.  Return the number written, or -1.

This function is a cancellation point and therefore not marked with
__THROW.  */
extern ssize_t write (int __fd, const void *__buf, size_t __n) __wur
__attr_access ((__read_only__, 2, 3));
```

我这里把在线文档上的内容也放到这里，可以对比起来看一下。

> read(2)<br/>
> read - read from a file descriptor<br/>
> #include <unistd.h><br/>
> ssize_t read(int fd, void *buf, size_t count);

> write(2)<br/>
> write - write to a file descriptor<br/>
> #include <unistd.h><br/>
> ssize_t write(int fd, const void *buf, size_t count);
