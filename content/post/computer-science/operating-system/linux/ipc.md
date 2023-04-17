---
draft: false
date: 2021-12-13 08:00:00 +0800
lastmod: 2022-01-24 08:00:00 +0800
title: "进程间通信（IPC）"
summary: "管道；匿名管道；命名管道；IPC；消息队列；信号量；共享内存；"
toc: true

categories:
- operating-system(操作系统)

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- linux-c
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- VMware Workstation Pro 16
- Ubuntu 22.04
- Linux 5.19.0-32-generic x86_64
- gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0

## 正文

### 进程间通信

进程间通信（Inter Process Communication，IPC）指的是在同一台机器上的不同进程之间传播或交换信息。

ipc 的方法主要包括 pipe（管道）和 ipc 对象（消息队列、信号量、共享内存）。管道包括匿名管道和命名管道。

ipc 对象有两个标准，一个是比较老的 "SYSTEM V" 标准的消息队列、信号量、共享内存，另一个是比较新的 POSIX 标准的消息队列、信号量、共享内存。

### 管道

> pipe(7)</br>
> Pipes and FIFOs (also known as named pipes) provide a unidirectional interprocess communication channel. A pipe has a read end and a write end. Data written to the write end of a pipe can be read from the read end of the pipe.</br>

管道提供了一个单向的进程间通信通道。一个管道有一个读取端和一个写入端。写入管道写入端的数据的数据可以从该管道的读取端读出。数据只能单向流动，数据从写入端流向读取端。

管道依赖 linux 内核实现，管道的本质是在内核中的一块内存（也叫内核缓冲区），这块缓冲区中的数据存储在一个环形队列中，由于管道在内核区，我们不能直接对其操作。

本质上是文件 I/O 操作。内核中管道两端分别对应两个文件描述符，通过写端的文件描述符把数据写入管道中，通过读端的文件描述符将数据从管道中读出来，读写管道的函数就是 linux 中的文件 I/O 函数。

管道分为匿名管道和命名管道。匿名管道，没有可以提供给外部的标识，所以，一般用于父子进程之间。命名管道，有可以提供给外部的标识，所以，可以用于非父子进程之间。

#### 匿名管道

> pipe(7)</br>
> A pipe is created using pipe(2), which creates a new pipe and returns two file descriptors, one referring to the read end of the pipe, the other referring to the write end. Pipes can be used to create a communication channel between related processes; see pipe(2) for an example.</br>

示例代码：{demo-c}/demo-in-linux/ipc/unnamed_pipe/unnamed_pipe.c

#### 命名管道

示例代码：

- 父子进程：{demo-c}/demo-in-linux/ipc/named_pipes/parent_child.c
- 非父子进程读取：{demo-c}/demo-in-linux/ipc/named_pipes/pipe_read.c
- 非父子进程写入：{demo-c}/demo-in-linux/ipc/named_pipes/pipe_write.c

```
> file fifo

fifo: fifo (named pipe)
```

可以使用 file 命令查看 fifo 的文件类型，可以看到这是个命名管道。

在 win10 环境的 docker 环境的 linux 环境中创建 fifo 文件时会出现 "Input/output error" 报错。这是因为 docker 目录是 win10 目录映射来的，创建不了 fifo 文件。解决办是用 linux 环境中的目录，测试的时候可以先用 "/tmp" 目录。

### IPC 对象

> ipc(2)</br>
> ipc() is a common kernel entry point for the System V IPC calls for messages, semaphores, and shared memory. call determines which IPC function to invoke; the other arguments are passed through to the appropriate call.</br>

ipc 对象包括消息队列、信号量、共享内存。创建 ipc 对象时需要指定一个 key（键，标识符）。key 是指 ipc 对象在操作系统内部的唯一标识，在多个进程通信时用作关联。

ipc 相关的 linux 命令：ipcs、ipcrm。

#### 消息队列

消息队列的实现依赖 linux 内核，并且它的数据存储在内核中。

示例代码：

- 创建：`demo_c/demo_linux_c/ipc/message_queue/msgget.c`
- 发送：`demo_c/demo_linux_c/ipc/message_queue/msgsnd.c`
- 接收：`demo_c/demo_linux_c/ipc/message_queue/msgrcv.c`
- 控制：`demo_c/demo_linux_c/ipc/message_queue/msgctl.c`

创建一个消息队列。

```
> ./msgget.elf
[debug]:mqId=0, errno=0, error=Success
```

消息队列创建完成后，可以通过 ipcs 命令查看，通过 key 字段可以找到创建的消息队列。向消息队列发送消息后，used-bytes 字段会显示消息队列使用的大小，messages 字段会显示未处理消息的条数。

```
> ipcs

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    
0x00001000 0          qqq        666        1024         0
```

发送一条消息。

```
> ./msgsnd.elf
[debug]:msgsndResult=0
```

查看消息队列的情况，可以看到，里面有一条消息。

```
> ./msgctl.elf
[debug]:msgctlResult=0, buf.msg_qnum=1
```

通过 ipcs 命令查看，可以看到，未处理消息的条数变成 1 了。

```
> ipcs

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    
0x00001000 0          qqq        666        1024         1
```

接收一条消息。

```
> ./msgrcv.elf
[debug]:msgrcvResult=1024, msg.mtext=hello, msg.mtype=1
```

查看消息队列的情况，可以看到消息已经被取走了。

```
> ./msgctl.elf
[debug]:msgctlResult=0, buf.msg_qnum=0
```

通过 ipcs 命令查看，可以看到，未处理消息的条数变成 0 了。

```
> ipcs

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    
0x00001000 0          qqq        666        0            0 
```

#### 信号量

信号量本质上是个计数器，linux 内核有相应的数据结构来维护。

注意，这里操作的是信号量集合，信号量集合里面可以由一个或多个信号量。

示例代码：

- 创建：`demo_c/demo_linux_c/ipc/semaphores/semget.c`
- 控制：`demo_c/demo_linux_c/ipc/semaphores/semctl.c`
- 操作：`demo_c/demo_linux_c/ipc/semaphores/semop.c`

创建有一个信号量的信号量集合：

```
> ./semget.elf
[debug]:semId=0, errno=0, error=Success
```

信号量创建完成后，可以通过 ipcs 命令查看，通过 key 字段可以找到创建的信号量。设置信号量后，nsems 字段对应信号量集中信号量的个数。

```
> ipcs

------ Semaphore Arrays --------
key        semid      owner      perms      nsems     
0x00001000 0          qqq        666        1 
```

设置和读取信号量集合里的信号量：

```
> ./semctl.elf
[debug]:SETVAL, semctlResult=0
[debug]:GETVAL, semctlResult=3
```

和消息队列的 msgctl 一样，信号量的 semctl 也有 IPC_STAT 、IPC_RMID 这两个参数，这里就不演示了。

操作信号量集合里的信号量：

```
> ./semop.elf
[debug]:SETVAL, semctlResult=0
[debug]:GETVAL, semctlResult=10
[debug]:semopResult=0
[debug]:GETVAL, semctlResult=8
```

一般最常用的信号量是二值信号量（0 和 1），用于进程间同步或是互斥操作。二值信号量主要特点就是集合里只有一个信号量，并且信号量的值只有 0 和 1 这两个。

对信号量进行加 1 叫做 V（vrijgeven），对它进行减 1 操作叫做 P（passeren）。和操作系统信号量的 pv 操作是一样的。二值信号量主要用于进程间同步，保证数据的一致性，对关键核心代码做临界，控制共享内存。

示例代码：`demo_c/demo-in-linux/ipc/semaphores/passeren_vrijgeven.c`

#### 共享内存

申请一块内存，进程间通过关联这块内存从而达到进程间通信的目的，它是 IPC 进程间通信中最快的。

示例代码：

- 创建：`demo_c/demo_linux_c/ipc/shared_memory/shmget.c`
- 连接、写入：`demo_c/demo_linux_c/ipc/shared_memory/shmat.c`
- 连接、读取、断开连接：`demo_c/demo_linux_c/ipc/shared_memory/shmdt.c`

创建共享内存：

```
> ./shmget.elf
[debug]:shmId=2, errno=0, error=Success
```

共享内存创建完成后，可以通过 ipcs 命令查看，通过 key 字段可以找到创建的共享内存。

```
> ipcs

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status      
0x00001000 2          qqq        666        1024       0 
```

连接共享内存、写入数据：

```
> ./shmat.elf
[debug]:shmId=2, errno=0, error=Success
```

连接共享内存、读取数据、断开连接：

```
> ./shmdt.elf
[debug]:shmId=2, errno=0, error=Success
[info]:msg=hello
[debug]:shmdtResult=0
```

## 参考

- {51CTO学堂}/{可用行师}/[Linux C核心技术](https://edu.51cto.com/course/28903.html)
  - 进程间通信部分
- {CSDN}/{富贵的编程日记}/[Linux快速入门之 管道（11）](https://blog.csdn.net/weixin_49730048/article/details/123595404)
- ChatGPT + DeepL