---
title: "operating system（操作系统）"
create_date: 2021-11-09 08:00:00 +0800
date: 2021-11-17 08:00:00 +0800
tags: computer-science operating-system
comment: false
show_author_profile: true
show_subscribe: false
---

- OS（operating system、操作系统）
  - batch processing（批处理）
  - device drivers（设备驱动程序）
  - multitasking（多任务处理）
    - scheduling（调度）
  - time-sharing（分时操作系统）
  - UNIX
    - kernel（内核）
      - memory management（内存管理）
      - multitasking（多任务）
      - I/O（输入输出处理）
    - runtime library（运行库）
    - panic（恐慌）
- interrupt（中断）
  - hardware interrupt（硬件中断）
  - soft interrupt（软中断）

### 操作系统

操作系统也是程序，但它有操作硬件的特殊权限，可以运行和管理其他程序。

批处理：一次输入多个程序，当运行完一个程序，会自动运行下一个程序。

### 中断

中断是一种异步的事件处理机制，可以提高系统的并发处理能力。

#### 硬中断

硬中断：操作系统用来响应硬件设备请求的一种机制。系统收到硬件的中断请求，会打断正在执行的进程，然后调用内核中的中断处理程序来响应请求。

操作系统收到了中断请求，会打断其他进程的运行。而且，中断处理程序在响应中断时，可能还会临时关闭中断。这意味着，如果当前中断处理程序没有执行完之前，系统中其他的中断请求都无法被响应，也就说中断有可能会丢失。

#### 软中断

Linux 系统为了解决中断处理程序执行过长和中断丢失的问题，将中断过程分成了两个阶段。

- 上半阶段：直接处理硬件请求（硬中断），一般会暂时关闭中断请求。主要负责处理跟硬件紧密相关或者时间敏感的事件。特点是快速执行。
- 下半阶段：由内核触发（软中断），一般以内核线程的方式延迟处理上半部未完成的工作。特点是延迟执行。

上半阶段相当于，内核先应答一声，但是有什么事等到了下半阶段再说。这个时候因为应答一声这个动作很快，所以即使暂时关闭中断请求，也会很快恢复，这中间一般不会发生中断丢失的问题。这个交给内核线程处理的下半部分，就是通常意义上的软中断。

软中断最常见的例子就是网卡收包。网卡收到网络包后，会通过硬中断通知内核有新的数据到了，于是内核就会调用对应的中断处理程序来响应该事件，这个事件的处理也是会分成上半部和下半部。

上半部分只要把网卡的数据读到内存中，然后更新一下硬件寄存器的状态，比如把状态更新为表示数据已经读到内存中的状态值。接着，内核会触发一个软中断，把一些处理比较耗时且复杂的事情，交给软中断处理程序去做，也就是中断的下半部分，其主要是需要从内存中找到网络数据，再按照网络协议栈，对网络数据进行逐层解析和处理，最后把数据送给应用程序。

在 Linux 中，

- 可以通过查看 /proc/softirqs 的内容来知晓软中断的运行情况。
- 可以通过查看 /proc/interrupts 的内容来知晓硬中断的运行情况。
- 但是这些数值是系统运行以来的累计中断次数，如果有需要应该关注中断次数的变化速率。
- 可以使用 `watch -d cat /proc/softirqs` 命令观察中断次数的变化速率。

一般对于网络 I/O 比较高的 Web 服务器，NET_RX 网络接收中断的变化速率相比其他中断类型快很多。如果发现 NET_RX 网络接收中断次数的变化速率过快，就可以使用 `sar -n DEV` 查看网卡的网络包接收速率情况，然后分析是哪个网卡有大量的网络包进来。然后就可以通过 `tcpdump` 抓包，分析这些包的来源。

### 参考

- Crash Course Computer Science（计算机科学速成课）
  - [bilibili](https://www.bilibili.com/video/BV1EW411u7th)
  - [CrashCourse 字幕组](https://github.com/1c7/crash-course-computer-science-chinese)
  - [Youtube 原视频](https://www.youtube.com/playlist?list=PL8dPuuaLjXtNlUrzyH5r6jN9ulI)
- [小林coding](https://xiaolincoding.com/)
  - [图解系统](https://xiaolincoding.com/os/)
