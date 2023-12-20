---
draft: false
date: 2023-05-20 08:00:00 +0800
title: "线程"
summary: "主线程；并发和并行；多线程；互斥锁；条件变量；惊群问题；"
toc: true

categories:
  - operating-system(操作系统)

tags:
  - computer-science(计算机科学)
  - operating-system(操作系统)
  - thread(线程)
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
[进程的创建、进程的运行、进程的内存资源、进程的退出、进程的回收](/计算机/operating-system/linux/process)

## 资料

- [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/thread/

## 正文

### 主线程

当一个进程启动时，默认会创建一个主线程。主线程会从 main() 开始执行代码。
main() 就是主线程的入口函数。每个线程都会有一个入口函数。
主线程一旦退出，整个进程就会退出，子线程无论有没有退出，都会被强制干掉。

### 并发和并行

当线程数量小于等于 CPU 核心数时，所有的线程并行执行。CPU 的核心够所有线程同时跑，不存在等待的情况。

当线程数量大于 CPU 核心数时，所有的线程并发执行。
CPU 的核心不够所有的线程同时跑，所以，每个线程执行一个时间片，然后，切换到其它线程执行。

### 多线程

多线程程序在执行的时候有两种情况。一种是子线程并入主线程，一种是主线程与子线程分离。

子线程并入主线程的情况，主线程调用 pthread_join() 后，会一直阻塞到子线程结束退出为止，当子线程结束退出后，主线程会继续往后执行。
主线程与子线程分离线的情况，主程调用 pthread_detach() 后，主线程与子线程分离，当子线程结束退出后，资源会被自动回收。

编写多线程程序的时候需要注意，如果主线程里面没有 sleep()、pthread_join()、pthread_detach() 这些相当于等待的代码结构。
那么，主线程执行完 pthread_create() 后就会结束，子线程没有来得急执行，就会被强制退出了。

代码示例：

- **{demo-c}/demo-in-linux/thread/thread_join.c**
- **{demo-c}/demo-in-linux/thread/thread_detach.c**

编译的时候，如果代码是多线程程序，gcc 命令需要加上 "\-lpthread" 选项。例如：`gcc xxx.c -lpthread -o xxx.elf`

线程可以通过调用 pthread_exit() 结束并退出，同时可以返回返回值。
比如，想返回个 3，这么写 "pthread_exit((void *)3)" 就行了。
或者直接用 return 也是可以的。同样的，想返回个 3，这么写 "return (void *)3" 就行了。

这样，pthread_join() 的第二个参数就能拿到返回值。注意，这个参数是个二级指针，一般传 **int 进去。

### 互斥锁

下面这段程序，本意是想将 x 中的数据取出来，对其 +100，然后再放回去。
如果跑两次的话，期望是依次输出 y=200，y=300。但是，在并发场景下，会输出 2 次 y=200。

因为，两个线程都会读到 100；然后，阻塞；然后，对其 +100；然后，赋值回去。

```
int x = 100;

void *mythread() {
    int y = x;
    sleep(1);
    y = y + 100;
    x = y;
    printf("[info]:child pthread, y=%d\n", x);
}
```

这里就需要用互斥锁对全局变量 x 进行保护，确保同一时刻只有一个线程有权利对 x 进行读写操作。

互斥锁常用于保护临界区的数据。有互斥锁的保护，在并发场景下，两个线程就会顺序执行，依次输出 y=200，y=300。

代码示例：**{demo-c}/demo-in-linux/thread/mutex_lock.c**

可以使用 pthread_mutex_init() 初始化互斥锁，也可以使用常量 PTHREAD_MUTEX_INITIALIZER 初始化互斥锁。

pthread_mutex_lock() 加不上锁的时候就会阻塞。对同一个锁重复执行加锁动作会死锁。

### 条件变量

程序在运行的时候会有，如果不满足某个条件就等待，直到条件满足再继续执行的场景。

假设，有一个线程 a 执行的时候，发现条件不满足，自己不能继续执行了，但是，自己前面加锁锁了临界资源 1。
再假设，有另一个线程 b，它手上有线程 a 需要的临界资源 2，但是，线程 b 需要线程 a 锁住的临界资源 1 才能继续执行。

这个时候，如果线程 a 不把所锁放掉，就陷入等待了，那么自己锁定的临界资源 1 就不会被释放。
这时，临界资源 1 是闲置的，但是，线程 b 就会拿不到，最终造成相互等待的死锁的情况。

这种情况就需要使用条件变量了。条件变量和需要结合互斥锁使用，互斥锁的存在是为了保护条件变量。
可以使用 pthread_cond_init() 初始化条件变量，也可以使用常量 PTHREAD_COND_INITIALIZER 初始化条件变量。

当线程 a 发现条件不满足时，就可以调用 pthread_cond_wait()。
调用 pthread_cond_wait() 会阻塞当前线程，并且会释放互斥锁。当 pthread_cond_wait() 被激活之后会再自动加锁。

另外的线程在释放掉临界资源后，可以使用 pthread_cond_signal() 唤醒一条阻塞的线程。
或者使用 pthread_cond_broadcast() 唤醒全部阻塞的线程。

代码示例：**{demo-c}/demo-in-linux/thread/cond_wait.c**

### 惊群问题

惊群问题（Thundering herd、cache stampede、cache-rashing），是多线程（多进程）程序中可能会发生的性能问题。

当多个线程（进程）竞争同一个共享资源，而该资源一次只能由一个对象访问时，就会出现惊群问题。

当资源可用时，所有的对象会同时醒来，并开始竞争同一个共享资源。这种突发的情况会导致资源使用量激增，进而导致系统性能下降。

惊群问题，可以使用互斥锁、信号量、或者其他同步原语，来调节对共享资源的访问。

缓存可以用来减少对共享资源的访问需求，提高系统的整体性能。负载均衡可以将共享资源分散出去，避免单点争夺。
