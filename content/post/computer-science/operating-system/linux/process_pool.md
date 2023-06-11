---
draft: false
date: 2023-05-17 08:00:00 +0800
title: "进程池"
summary: "进程池；"
toc: true

categories:
  - operating-system(操作系统)

tags:
  - computer-science(计算机科学)
  - operating-system(操作系统)
  - linux
  - c-programming-language
  - process(进程)
---

## 前言

实践的环境：

- amd64（x86_64）
- windows 11
- vmware workstation pro 16
- ubuntu 22.04
- linux version 5.19.0-41-generic
- gcc version 11.3.0

前置笔记：

- [进程的创建、进程的运行、进程的内存资源、进程的退出、进程的回收](/post/computer-science/operating-system/linux/process)
- [信号](/post/computer-science/operating-system/linux/signal)
- [进程间通信（IPC）](/post/computer-science/operating-system/linux/ipc)

## 资料

- [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/process-pool/

## 正文

### 进程池

进程的创建和销毁是有代价的。每次需要的时候，新创建一个子进程，使用完了在销毁掉，这种方式非常的浪费资源。

进程池就是预先创建好一组用于处理某种问题的子进程，等遇到这个问题的时候直接就可以丢给其中一个子进程去处理。

使用完了并不会销毁掉子进程，而是把数据都初始化回去之后，重新放回进程池中。这样就可以避免频繁的创建和销毁子进程。

实现进程池的时候需要注意几点。

- 父进程怎么给子进程传递数据。
- 父进程怎么选择用哪个子进程。
- 子进程用完了之后要重新初始化。
- 子进程怎么退出，防止变成孤儿进程。
- 父进程怎么接收外部数据。

代码示例：

- **{demo-c}/demo-in-linux/process-pool/pool.c**
- **{demo-c}/demo-in-linux/process-pool/write.c**

示例中：

- 父进程通过匿名管道传递数据给子进程。
- 子进程的工作是接收父进程的数据并打印。
- 父进程使用轮询算法选择用哪个子进程。
- 子进程用完了之后会重新初始化接收缓冲区。
- 子进程通过父进程发送的信号退出。
- 父进程通过命名管道从外部接收数据。
- 父进程通过信号和特殊的输入 "exit" 退出。

## 参考

- {51CTO学堂}/{可用行师}/[Linux C核心技术](https://edu.51cto.com/course/28903.html)
    - 核心基础的，进程池部分；
