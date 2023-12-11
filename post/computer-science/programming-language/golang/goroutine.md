---
draft: false
date: 2022-05-16 08:00:00 +0800
lastmod: 2022-05-16 08:00:00 +0800
title: "goroutine"
summary: "goroutine"
toc: true

categories:
- golang

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- golang
---

### 并行和并发

- 并行：多个程序在多个核心同时跑
- 并发：多个程序看上去在同时跑

### 进程

操作系统会为应用程序创建一个进程。进程会包含应用程序运行的资源，如：线程、内存、文件描述符等。

一个进程从主线程开始，当该线程终止时，进程终止。线程可以启动更多的线程。

线程是操作系统调度的一种执行路径，用于在处理器执行我们在函数中编写的代码。

操作系统调度线程在可用处理器上运行，Go 运行时调度 goroutine 在绑定到单个操作系统线程的逻辑处理器（P）中运行。

### goroutine

使用 goroutine 的两个注意点：

- 什么时候结束
- 有没有强制结束的手段

如果函数启动 goroutine，则必须向调用方提供显式停止该 goroutine 的方法。

并发环境下，要保证只有写的一方可以关闭 chan。

### 串行化

- sync
  - Mutex
  - RWMutex
- sync/atomic
- channel

### happen-before

编译器和 CPU 在不改变 goroutine 的行为前提下，为了更高的性能可能修改读和写的执行顺序。由于存在重排，`a=1`、`b=2` 两条语句，在两个 goroutine 中可能会看到不同的执行顺序。

并发不加锁的场景下，`i++` 都不一定能得到争取的结果，`i++` 最终在执行的时候会变成 3 条汇编指令。并发不加锁时，多个 CPU 核心的 CPU 高速缓存和内存会出现类似缓存不一致的问题。

### machine word

对于 single machine word 的变量的读写操作应该是原子的。比如 64 位的机器，把 int 指针从 int a 指向 int b 这个操作应该是原子的。如果是把 20 字节的字符串从 a 赋值给 b，因为一次最多处理 64 位的数据，所以这个赋值操作可能存在中间状态。

### 数据争用（data race）和竞态条件（race condition）

- `go build -race`
- `go test -race`

检测到数据争用之后，程序会直接停止。

没有安全的数据争用（safe data race）。程序要么没有数据争用，要么未定义原子性和可见性。

给 slice、map、interface 赋值，不是原子操作。比如 interface 底层是两个指针。

### 几种锁的实现

- barging：持有锁的 goroutine 释放锁后，唤醒等待队列的第一个 goroutine，然后就不管了。锁会被等待队列的第一个 goroutine 或者第一个请求锁的 goroutine 获取。
- handsoff：持有锁的 goroutine 释放锁后，会一直持有锁，直到把锁交给等待队列的第一个 goroutine。
- spinning（自旋）：获取锁的 goroutine 获取不到锁时，会原地重试几次。

- barging 可以提高吞吐量，但是会产生锁饥饿问题。
- handsoff 可以解决锁饥饿问题，但是效率不高。
- spinning 会占用 CPU 资源。但是相比 goroutine 的 parking 和 unparking，自旋要更快。

Go 1.8 使用了 Barging 和 Spining 的结合实现。当试图获取已经被持有的锁时，如果本地队列为空并且 P 的数量大于1，goroutine 将自旋几次（用一个 P 旋转会阻塞程序）。自旋后，goroutine park。

Go 1.9 通过添加一个新的饥饿模式来解决先前解释的问题，该模式将会在释放时候触发 handsoff。所有等待锁超过一毫秒的 goroutine（也称为有界等待）将被诊断为饥饿。当被标记为饥饿状态时，unlock 方法会 handsoff 把锁直接扔给第一个等待者。在饥饿模式下，自旋也被停用，因为传入的 goroutine 将没有机会获取为下一个等待者保留的锁。

- waitgroup
- errgroup
- sync.Poll

### channel（通道）

- 无缓冲通道：读写必须都准备好，否则阻塞
- 有缓冲通道：写时，可以缓存一些数据，满则阻塞。读时，空则阻塞。

无缓冲通道或缓冲太小，导致 goroutine 频繁阻塞和唤醒，会消耗性能。但是有缓冲通道用空间换时间也有极限。

向一个已经关闭的通道写数据会 panic。

常用模型：

- timing out
- moving on
- pipeline
- fan-out，fan-in
- cancellation

### context

context 可用于管控 goroutine 生命周期，级联取消可用于快速取消整条链上的 goroutine。

context 是请求级别的，只有和请求有关系的数据才可以挂到上下文中使用。

在需要向 context 添加数据时，应该先深拷贝一份，然后修改数据继续向下传递。因为别的 goroutine 可能在并发访问这个 context 。