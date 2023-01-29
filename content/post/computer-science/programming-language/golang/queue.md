---
draft: false
date: 2023-01-29 08:00:00 +0800
lastmod: 2023-01-29 08:00:00 +0800
title: "使用 Golang 实现并发安全队列"
summary: "锁实现，CAS+自旋实现"
toc: true

categories:
- golang

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- golang
- concurrent(并发)
- queue(队列)
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> go version go1.19 windows/amd64

### 资料

- [{demo-golang}](https://github.com/KelipuTe/demo-golang)/demo/queue/
- <a href="/drawio/computer-science/programming-language/golang/queue.drawio.html">queue.drawio.html</a>

### 前言

并发问题一定要从指令层面保证不会出问题，而不是语句层面。例如一个简单的 `i++` 语句，在指令层面至少分为三步：加载变量 i，计算 i+1 的结果，把 i+1 的结果赋值给变量 i。高并发场景下，执行 `i++` 语句的线程，有可能在任意时刻被剥夺 CPU 资源。这种情况会导致语句实际的结果和预期的结果不一致。

假设 i=0，这时线程 a 和线程 b 同时执行 `i++` 语句。这里预期的结果应该是 i=2，但是实际执行过程中会发生下面这种情况。线程 a 先加载变量 i，拿到 0，然后执行完"计算 i+1 的结果（结果为 1）"后，没来得及执行"把 i+1 的结果（结果为 1）赋值给变量 i"的操作，就被剥夺了 CPU 资源给到线程 b。

因为线程 a 还没有写回数据，所以线程 b 第一步拿到的 i 依然是 0。假设线程 b 没有被剥夺 CPU 资源，完整地跑完了三条指令，那么 i 应该是 1。线程 b 结束运行，CPU 资源给到线程 a 继续执行。这个时候线程 a 执行最后一步，把 i+1 的结果（结果为 1）赋值给变量 i。这样就出问题了，实际的结果 i=1 和期望的结果 i=2 不一致。

### 并发阻塞队列（锁实现）

代码详见：demo-golang/demo/queue/wait_cond.go。

加锁可以在并发环境下保护队列数据的正确性。但是如果 gorouting 里面单纯的对 sync.Mutex 进行加锁操作。gorouting 就陷进去了，直到它拿到锁，否则是不会出来的。也就是说，单纯地加锁无法控制超时。

用 sync.Cond 和 sync.Mutex 区别不是很大。执行 Wait() 之后，gorouting 一样会陷进去，直到它被 Signal() 或者 Broadcast() 发出的信号唤醒。

所以这里需要的是一个可以被控制的等待加锁的结构。也就是说要一个 sync.Mutex.Luck(time) 或者 sync.Cond.Wait(time) 这样的东西。如果一定时间拿不到锁，要能从阻塞状态退出来。

可以先看一眼官方提供的 sync.Cond.Wait() 里面的逻辑。执行逻辑很简单，大概是：获取一个用于等通知的结构；把拿着的锁放掉；阻塞，等通知；等到通知了，把锁加回来。

```
func (c *Cond) Wait() {
	c.checker.check()
	t := runtime_notifyListAdd(&c.notify)
	c.L.Unlock()
	runtime_notifyListWait(&c.notify, t)
	c.L.Lock()
}
```

加锁和解锁的步骤是必须的，所以核心问题其实就找到了。需要修改原有的等通知的逻辑，让这里变成可以被 context 超时控制的结构。在 golang 里面管道可以变相的做到可以被阻塞也可以在需要的时候被唤醒。可以把读一个空的管道理解成阻塞，把从管道里读到东西理解成唤醒。

如果需要模拟阻塞，那么就读一个空的管道就可以了。唤醒这里需要一点技巧，这里不能用往管道里写数据的方式去唤醒阻塞中的 gorouting。因为不知道到底有多少个 gorouting 在等着读数据，也就是不知道要写多少次数据。写少了，会有等着的 gorouting 读不到数据，会泄露。写多了自己会阻塞，也会泄露。

这里可以用直接关闭管道的思路，这样所有的尝试读取管道的 gorouting 都会读到零值。相当于完成了一次 Broadcast()。但是这种操作无法实现 Signal()。

### 并发非阻塞队列（CAS+自旋实现）

CAS（Compare And Swap）操作是原子操作，包含三个操作数：内存中的值（内存位置 V）、原值（预期原值 A）、新值（新值 B）。如果内存中的值与原值相等，那么处理器会将内存中的值更新为新值。如果内存中的值与原值不相等，处理器不做任何操作。无论哪种情况，都会在 CAS 指令之前返回内存中的值。
