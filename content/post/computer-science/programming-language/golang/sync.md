---
draft: false
date: 2023-01-14 08:00:00 +0800
lastmod: 2023-01-14 08:00:00 +0800
title: "Golang 的 sync 包的使用"
summary: "sync.Mutex，sync.RWMutex，sync.Cond，WaitGroup"
toc: true

categories:
- golang

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- golang
- concurrent(并发)
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> go version go1.19 windows/amd64

### 资料

- [{demo-golang}](https://github.com/KelipuTe/demo-golang)/demo/sync/

### 调度器

代码见：{demo-golang}/demo/sync/goroutine_test.go

调度器在需要的时候，只会对正在运行的 goroutine 发出通知，试图让它停下来。但是调度器不会也不能强行让一个 goroutine 停下来。所以如果循环中的语句过于简单的话，那么当前的 goroutine 就可能不会正常响应（或者说没有机会响应）调度器的停止通知。

### sync.Mutex

sync.Mutex 就是互斥锁或者互斥量。sync.Mutex 使用过程中有几点需要注意：不要重复加锁、不要忘记解锁、不要对未锁定的锁解锁、不要对已解锁的锁解锁。

另外，sync.Mutex 是一个结构体类型，属于值类型中的一种。所以不要在多个函数之间直接传递。把它传给一个方法、将它从方法中返回、把它赋给其他变量、让它进入某个 channel，都会导致副本的产生。原值和它的副本，以及多个副本之间都是完全独立的，它们都是不同的 sync.Mutex。

### sync.RWMutex

sync.RWMutex 就是读写锁，可以单独加读锁或者写锁，其余的和 sync.Mutex 差不多。

### sync.Cond

代码见：{demo-golang}/demo/sync/cond_test.go

sync.Cond 可以理解为条件变量，需要结合 sync.Mutex 或者 sync.RWMutex 一起使用。用于控制 goroutine 的动作。不满足条件时，阻塞 goroutine。满足条件时，唤醒一个阻塞中的 goroutine 或者唤醒全部阻塞中的 goroutine。

sync.Cond 主要有三个方法：Wait()、Signal()、Broadcast()

Wait() 会阻塞当前 goroutine，并当前的 goroutine 添加到通知队列的队尾，直到通知到来。阻塞时，会解锁当前条件变量基于的互斥锁。通知到来时，会唤醒当前 goroutine，重新锁定当前条件变量基于的互斥锁。

Signal() 从通知队列的队首开始，找第一个符合条件的 goroutine 唤醒。而 Broadcast() 则会唤醒通知队列所有符合条件的 goroutine。

另外，在使用时，应该使用 for 语句包裹条件变量的 Wait() 方法。这里有两个典型场景。

场景1：并发环境下，可能有多个 goroutine 在等待同一个共享资源。如果一个 goroutine 被唤醒后，发现拿不到共享资源，那么就应该再次调用 Wait()，继续等待。

场景2：需要的共享资源数量大于 1 个，也就是说只拿到一个共享资源是不满足条件的。这种情况下 goroutine 被唤醒后，需要先检查可以被占用的共享资源能不能满足自己的需求。如果不能满足需求，那么就应该再次调用 Wait()，继续等待。

比如需要的共享资源数量是 2。假设，现在有 goroutine0 到 0 个共享资源，goroutine2 拿到 1 个共享资源。这两个 goroutine 都不满足条件，都在等待。这个时候，goroutine4 释放了 1 个共享资源。

如果用的是 Signal()，假设它唤醒了 goroutine0，这个时候就算占用了 goroutine4 释放的 1 个共享资源，也是不能满足要求的，所以还是需要继续等待。不如唤醒 goroutine2，它就缺一个。所以这种情况就可以使用 Broadcast() 通知所有等待中的 goroutine。

### WaitGroup

代码见：{demo-golang}/demo/sync/wait_group_test.go

WaitGroup 可以结合 channel 控制限制开启的 gorouting 数量。

用 channel 模拟令牌桶的思路，每次循环在开启 gorouting 之前需要先获取到令牌。gorouting 执行完之后，需要归还令牌，然后循环就可以继续创建新的 gorouting 了。
