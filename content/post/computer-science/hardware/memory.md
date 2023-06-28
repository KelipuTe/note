---
draft: false
date: 2023-02-21 08:00:00 +0800
lastmod: 2023-02-21 08:00:00 +0800
title: "存储器（Memory）"
summary: "锁存器；门锁；寄存器；可寻址内存；"
toc: true

categories:
- hardware(硬件)

tags:
- computer-science(计算机科学)
- hardware(硬件)
---
## 前言

前置笔记：[算术逻辑单元（Arithmetic and Logic Unit、ALU）](/post/computer-science/hardware/alu)

## 资料

- <a href="/drawio/computer-science/hardware/memory.drawio.html">memory.drawio.html</a>

## 正文

### 锁存器

锁存器（and-or latch）由与门、或门、非门组成。（见图：**memory.drawio.html 2-2**）可以存储 1 bit 的数据。

初始状态，set 线不通电，reset 线不通电（见图：**memory.drawio.html 2-4-2**），此时输出 0。
没有输入，整个结构也能持续输出 0，换句话说就是锁存器存储了 0。

当 set 线通电，reset 线不通电时（见图：**memory.drawio.html 2-4-4-2**），此时输出 1。
如果这个时候 set 线从通电变成不通电（见图：**memory.drawio.html 2-4-4-4**），
没有输入，整个结构依然能持续输出 1，换句话说就是锁存器存储了 1。

想要把锁存器里面恢复成 0，只需要 reset 线通电即可。这个时候 set 线通不通电是无所谓的，都会输出 0。
（见图：**memory.drawio.html 2-4-6-2、2-4-6-4、2-4-8-2、2-4-8-2**）

### 门锁

锁存器已经可以实现存储 1 bit 的数据了，但是它有一点不好用。
第一，set 线用于输入 1，reset 线用于输入 0，输入数据需要两条线。
第二，没有安全机制，任意一条线通电了，就有可能改变里面存储的数据。

可以在锁存器的前面加上一些限制结构解决前面的两个问题，然后就可以得到 gated latch（门锁）（见图：**memory.drawio.html 4-2**）。

初始状态，data input 线不通电，write enable 线不通电（见图：**memory.drawio.html 4-4-2**），此时输出的是 0。

如果只给 data input 线通电，而 write enable 线不通电的话，1 这个数据是设置不进去的（见图：**memory.drawio.html 4-4-4**）。
只有当 data input 和 write enable 线都通电时，1 这个数据才能设置进去（见图：**memory.drawio.html 4-4-6-2、4-4-6-4**）。

想要把门锁里面恢复成 0，就需要给 write enable 线通电的同时保证 data input 线不通电，
这里就和锁存器那里 set 线通不通电无所谓不一样了（见图：**memory.drawio.html 4-4-8-2、4-4-8-4**）。

门锁可以做到一条线控制输入的数据，另一条线控制允不允许输入，操作的逻辑非常清晰。

### 寄存器

把 n 个门锁线性排列起来，就可以存储 n 个 bit 的数据了，寄存器（register）就是这种结构。
这里简单画一个 8 位的寄存器。（见图：**memory.drawio.html 6-2**）

寄存器的位宽指的是，寄存器能存储多少位数据。

### 二维的平面结构

如果需要存储的数据容量很大的话，寄存器这种一维的线性的结构就不合适了，导线数量太多。
所以需要排列更紧密的结构，让导线可以复用，从而减少导线的数量。比如，二维的平面结构，或者三维的立体结构。
但是这会带来新的问题，怎么找到众多门锁中的某一个门锁？这里用二维的平面结构举例。

门锁是没有位置信息的，但是二维的平面结构是可以放到坐标系里去的，这样二维的平面结构里的门锁就可以用行列坐标表示了。
把行列坐标和 write enable 线组合起来，就可以控制二维的平面结构里的某一个门锁了。
用同样的思路，还可以控制某一个门锁的输出，也就是图里的 read enable 线。
read enable 线结合二极管就可以把数据的输入输出都用一条线解决。（见图：**memory.drawio.html 8-2**）

这里简单画一下存入数据和读取数据的示意图。（见图：**memory.drawio.html 8-4-2、8-4-4**）

### 可寻址内存

这里把一个 2 行 2 列的二维的平面结构画出来（见图：**memory.drawio.html 8-8-2**）。
显然还有一个问题，行列的电路怎么控制？这里就要用到"多路复用器"结构。

多路复用器就是一堆不同逻辑门的组合，用于判断输入是不是满足某个条件。这里画一个输入为 2 位二进制树的多路复用器。
（见图：**memory.drawio.html 8-8-4**）举例，输入 0b10 的时候，多路复用器只有 01 对应的那个输出端会有电。

把多路复用器加到图里，行列两个方向上都放上一个多路复用器，（见图：**memory.drawio.html 8-8-6**）这样就可以控制所有的存储单元了。
举例，输入行 0b10、列 0b11 的时候，对应到图中，就是第 2 行、第 3 列的那个单元。

2 行 2 列的可以得到一个 16 bit 的内存。再扩展一下，从 2 行 2 列变成 16 行 16 列，
就可以得到一个 256 bit 的内存（见图：**memory.drawio.html 8-8-10**），每个内存地址只能存储 1 bit 的数据。

现在常用的内存，一个内存地址可以存储一个字节，也就是 8 bit 的数据，这是怎么做到的？这里用到的思路和寄存器那里是一样的，
把 8 个 256 bit 的内存线性排列起来（见图：**memory.drawio.html 8-10-2**），让一个地址对应 8 块内存里相同的位置。

假设，有一个字节的数据 0b10001000 要存储进去。这里用到的思路是拆开，把 0b10001000 拆成 1、0、0、0、1、0、0、0 分别存储。
第 0 位，放到第一块；第 1 位，放到第二块；以此类推。这样就可以让一个地址对应 8 块内存里相同的位置都存储了这个数据的一部分。
取数据的时候，把取出来的 8 个 1 bit 的数据，按照拆分规则再组合起来就可以了。

可寻址内存的一个重要特性就是可以随时访问任意位置。
可寻址内存的地址从 0 开始编号，然后，自增排列。
因为内存是线性结构而且有地址，所以理论上读写内存上任何一个数据的速度是一样的。

不同的 RAM 用不同的存储单元（电路）存取单个 bit，比如：
不同的逻辑门、capcitor（电容器）、charge traps（电荷捕获）、memristor（记忆电阻器）。

## 参考

- Crash Course Computer Science（计算机科学速成课）
    - [bilibili](https://www.bilibili.com/video/BV1EW411u7th)
    - [CrashCourse 字幕组](https://github.com/1c7/crash-course-computer-science-chinese)
    - [Youtube 原视频](https://www.youtube.com/playlist?list=PL8dPuuaLjXtNlUrzyH5r6jN9ulI)
    - 6、寄存器 & 内存-Registers and RAM
