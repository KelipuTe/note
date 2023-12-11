---
draft: false
date: 2023-02-21 08:00:00 +0800
title: "CPU 高速缓存"
summary: "CPU 高速缓存；"
toc: true

categories:
  - 硬件

tags:
  - 计算机科学
  - 硬件
---

## 反向链接

[CPU](/post/computer-science/hardware/CPU)；
[可寻址内存](/post/computer-science/hardware/可寻址内存)；
[速度差距](/post/computer-science/速度差距)；
[局部性原理](/post/computer-science/局部性原理)；

## 正文

### CPU 高速缓存

- CPU 高速缓存、CPU cache；
- 缓存块、cache line、cache block；

CPU 高速缓存就是在 CPU 里放一个小容量的存储器用来暂存内存中的数据。

高速缓存的工作原理是基于局部性原理的。依据局部性原理，高速缓存从 RAM 拿数据时，
一次拿一个缓存块大小的数据而不是一个字节大小的数据。

### 一级缓存

一级缓存（L1 Cache）可分为指令缓存（I-Cache）和数据缓存（D-Cache）。每个 CPU 核心都有属于自己的一级缓存。

在 Linux 中：

- /sys/devices/system/cpu/cpu0/cache/index0/coherency_line_size 可以查看 L1 缓存块的大小。
- /sys/devices/system/cpu/cpu0/cache/index0/size 可以查看 L1 数据缓存的大小； 
- /sys/devices/system/cpu/cpu0/cache/index1/size 可以查看 L1 指令缓存的大小。

### 二级缓存

每个 CPU 核心都有属于自己的二级缓存（L2 Cache）。

在 Linux 中，/sys/devices/system/cpu/cpu0/cache/index2/size 可以查看 L2 的大小。

### 三级缓存

三级缓存（L3 Cache）通常是多个 CPU 核心共用。

在 Linux 中，/sys/devices/system/cpu/cpu0/cache/index3/size 可以查看 L3 的大小。

### 映射方式

### 缓存命中率（cache hit rate）

缓存命中、cache hit；缓存未命中、cache miss；

- 数据缓存的命中率：尽量顺序访问数据。
- 指令缓存的命中率：尽量访问有序的数据（让分支预测更准确）。

多核 CPU 缓存命中率：把计算密集型程序的线程绑定在某个 CPU 核心上。
对于多核心 CPU，线程可能在不同 CPU 核心来回切换，这对属于每个核心的 L1 和 L2 不利，但是对 L3 没影响。

在 Linux 中，提供了 sched_setaffinity，来实现将线程绑定到某个 CPU 核心这一功能。

### 缓存一致性

- 缓存一致性、cache coherence；
- 写直达、write through；写回、write back；

因为高速缓存是用来暂存内存中的数据的。
一个数据出现在了两个地方，这就存在一致性问题。

缓存一致性有两个解决方案：写直达、写回。

#### 写直达

当发生写操作时，把数据同时写入缓存和内存。

CPU 写入前先判断缓存里有没有数据。
如果缓存里有，就先更新到缓存，再写入内存。
如果缓存里没有，就直接更新到内存。

#### 写回

当发生写操作时，新的数据只被写入到缓存块里，只有当修改过的缓存块被替换时，才需要写到内存中。

这里又有目标数据再缓存里和目标数据不在缓存里两种情况。

如果数据在缓存里，则把新的数据写入到缓存块里，同时标记缓存块为脏（Dirty）的。
这个脏的标记代表这个时候缓存里面的数据和内存中的数据是不一致的。

如果要写入新的数据的缓存块里存放的是别的内存地址的数据，就要检查这个缓存块有没有被标记为脏的。

如果是脏的话，要把这个缓存块里的数据写回到内存。
然后把当前要写入的数据先从内存读入到缓存里，接着把数据写入到缓存块，最后把它标记为脏的。

如果不是脏的话，就直接将数据写入到这个缓存块里，然后把这个缓存块标记为脏的。

### 多核心缓存一致性

多核心 CPU 的每个核心都有自己的 L1 和 L2。写直达和写回方案，只能解决单核心的问题。

要解决多核心的问题，需要做到下面两点：

- 写传播（wreite propagation）：某个核心里的缓存更新时，必须传播到其他核心。用于解决下面的情况 1。
- 事务串形化（transaction serialization）：某个核心里对数据的操作顺序，在其他核心看起来必须是一样的。用于解决下面的情况 2。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/hardware/wreite_propagation.drawio.png">
</div>

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/hardware/transaction_serialization.drawio.png">
</div>

总线嗅探（bus snooping）：当某个核心更新了缓存，要把该事件广播到其他核心。核心需要每时每刻监听总线上的一切活动，不管别的核心的缓存是否缓存相同的数据，都需要发出一个广播事件。

总线嗅探并不能保证事务串形化。要实现事务串形化，需要做到下面两点：

- CPU 核心对于缓存中数据的操作，需要同步给其他 CPU 核心。
- 引入锁的概念，如果两个 CPU 核心里有相同数据的缓存，那么只有拿到了锁，才能进行对应的数据更新。

### MESI 协议

基于总线嗅探机制实现了事务串形化，同时利用状态机机制降低了总线带宽压力。

MESI 是 4 个单词的缩写：Modified（已修改）、Exclusive（独占）、Shared（共享）、Invalidated（已失效）。

- 已修改状态就是前面提到的脏标记，代表缓存块上的数据已经被更新，但还没有写到内存。
- 已失效状态表示的是这个缓存块里的数据已经失效了，不可以读取该状态的数据。
- 独占和共享状态都代表缓存块里的数据是干净的，和内存里面的数据是一致性的。

- 独占状态的时候，数据只存储在一个 CPU 核心的缓存里，而其他 CPU 核心的缓存没有该数据。这个时候，不存在缓存一致性的问题。所以可以直接自由地写入，而且不需要通知其他 CPU 核心。
- 在独占状态下的数据，如果有其他核心从内存读取了相同的数据到各自的缓存，那么这个时候，独占状态下的数据就会变成共享状态。
- 共享状态代表着相同的数据在多个 CPU 核心的缓存里都有，所以当要更新缓存里面的数据的时候，不能直接修改，而是要先向所有的其他 CPU 核心广播一个请求，要求先把其他核心的缓存中对应的缓存块标记为已失效状态，然后再更新当前缓存里面的数据。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/hardware/cache_mesi.drawio.png">
</div>