---
draft: false
date: 2023-02-20 08:00:00 +0800
lastmod: 2023-02-20 08:00:00 +0800
title: "Memory（内存）"
summary: "锁存器；门锁；寄存器；内存；"
toc: true

categories:
- hardware(硬件)

tags:
- computer-science(计算机科学)
- hardware(硬件)
---

## 资料

- <a href="/drawio/computer-science/hardware/memory.drawio.html">memory.drawio.html</a>

## 正文

### 锁存器

and-or latch（锁存器）由与门、或门、非门组成（见图：**memory.drawio.html 2-2**）。可以存储 1 bit 的数据。

初始状态，set 线不通电，reset 线不通电（见图：**memory.drawio.html 2-4-2**），此时输出的是 0，也可以认为锁存器存储了 0。

当 set 线通电，reset 线不通电时（见图：**memory.drawio.html 2-4-4-2**），此时输出变成 1。如果这个时候 set 线从通电变成不通电（见图：**memory.drawio.html 2-4-4-4**），那么整个结构依然可以输出 1，相当于这个 1 就被锁存器存下来了。

想要把里面存住的 1 恢复成 0，只需要 reset 线通电即可。这个时候 set 线通不通电是无所谓的，都会输出 0（见图：**memory.drawio.html 2-4-6-2、2-4-6-4、2-4-8-2、2-4-8-2**）。

### 门锁

锁存器已经可以实现存储 1 bit 的数据了，但是它有一点问题。set 用于输入 1，reset 用于输入 0，输入数据需要两条线。另外，没有安全机制，任意一条线通电了，就有可能改变里面存储的数据。所以可以在锁存器的前面，加上与门、非门进行限制，然后就可以得到 gated latch（门锁）（见图：**memory.drawio.html 4-2**）。

初始状态，data input 线不通电，write enable 线不通电（见图：**memory.drawio.html 4-4-2**），此时输出的是 0，也可以认为门锁存储了 0。

如果只给 data input 线通电，而 write enable 线不通电的话，1 这个数据是设置不进去的（见图：**memory.drawio.html 4-4-4**）。只有当 data input 和 write enable 线都通电时，1 这个数据才能设置进去（见图：**memory.drawio.html 4-4-6-2、4-4-6-4**）。

想要把里面存住的 1 恢复成 0，就需要给 write enable 线通电的同时保证 data input 线不通电，这里就和锁存器那里 set 线通不通电无所谓不一样了（见图：**memory.drawio.html 4-4-8-2、4-4-8-4**）。

这样就可以做到一条线控制输入的数据，另一条线控制允不允许输入了，这个操作的逻辑是非常清晰的。

### 寄存器

把 n 个门锁线性排列起来，就可以存储 n 个 bit 的数据了，这种结构叫 register（寄存器）。这里简单画一个 8 位的寄存器（见图：**memory.drawio.html 6-2**）。

### 二维的平面结构

如果需要存储的数据容量很大的话，寄存器这种一维的线性的结构就不合适了，需要排列更紧密的结构。比如，二维的平面结构，或者三维的立体结构。但是这会带来新的问题，怎么找到众多门锁中的某一个门锁？这里用二维的平面结构举例。

门锁是没有位置信息的，但是二维的平面结构是可以放到坐标系里去的，这样二维的平面结构里的门锁就可以用行列坐标表示了。把行列坐标和 write enable 线组合起来，就可以控制二维的平面结构里的某一个门锁了（见图：**memory.drawio.html 8-2**）。

用同样的思路，还可以控制某一个门锁的输出，也就是图里的 read enable 线。read enable 线结合二极管就可以把数据的输入输出都用一条线解决。这里简单画一下存入数据和读取数据的图（见图：**memory.drawio.html 8-4-2、8-4-4**）。

### 内存

这里把一个 2 行 2 列的二维的平面结构画出来（见图：**memory.drawio.html 8-8-2**）。显然还有一个问题，行列的电路怎么控制？这里就要用到图里的 "多路复用器" 结构。

多路复用器就是一堆不同逻辑门的组合，用于判断输入是不是满足某个条件。比如，输入 0b10 的时候，多路复用器会输出 2。对应到图中，就是标记为 2 的那条线通电了。行列两个方向上都放上一个多路复用器，就可以控制所有的存储单元了。

这里就得到了一个 16 bit 的内存。再扩展一下，从 2 行 2 列变成 16 行 16 列，就可以得到一个 256 bit 的内存（见图：**memory.drawio.html 8-8-4**）。

上面的那块 256 bit 的内存，每个内存地址只能存储 1 bit 的数据。现在常用的内存，一个内存地址可以存储一个字节，也就是 8 bit 的数据，这是怎么做到的？这里用到的思路和寄存器那里是一样的，把 8 个 256 bit 的内存线性排列起来（见图：**memory.drawio.html 8-10-2**），让一个地址对应 8 块内存里相同的位置。

假设，有一个字节的数据 0b10001000 要存储进去。这里用到的思路是拆开，把 0b10001000 拆成 1、0、0、0、1、0、0、0 分别存储。第 0 位，放到第一块；第 1 位，放到第二块；以此类推。这样就可以让一个地址对应 8 块内存里相同的位置都存储了这个数据的一部分。取数据的时候，把取出来的 8 个 1 bit 的数据，按照拆分规则再组合起来就可以了。

## reference（参考）

- Crash Course Computer Science（计算机科学速成课）
    - [bilibili](https://www.bilibili.com/video/BV1EW411u7th)
    - [CrashCourse 字幕组](https://github.com/1c7/crash-course-computer-science-chinese)
    - [Youtube 原视频](https://www.youtube.com/playlist?list=PL8dPuuaLjXtNlUrzyH5r6jN9ulI)
