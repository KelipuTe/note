---
draft: false
date: 2023-02-22 08:00:00 +0800
lastmod: 2023-02-22 08:00:00 +0800
title: "中央处理器（Central Processing Unit、CPU）"
summary: "程序；指令；指令集；控制单元；CPU；CPU 执行程序的过程；"
toc: true

categories:
- hardware(硬件)

tags:
- computer-science(计算机科学)
- hardware(硬件)
---
## 前言

前置笔记：

- [算术逻辑单元（Arithmetic and Logic Unit、ALU）](/post/computer-science/hardware/alu)
- [内存（Memory）](/post/computer-science/hardware/memory)

## 资料

- <a href="/drawio/computer-science/hardware/cpu.drawio.html">cpu.drawio.html</a>

## 正文

### 程序

ALU 可以进行算数运算，内存可以存储数据。如果能把两个结合起来，那就可以处理复杂的逻辑了。ALU 有三个输入，这里假设有两个 8 bit 的二进制数和一个操作码，但是一个内存地址只能存储 8 bit，显然不可能把所有的东西都存在一个内存地址上，然后一股脑的扔给 ALU。

所以这里就需要一种方式把需要执行的逻辑转化成一个一个的小步骤，每个内存地址存一个小步骤，然后依次执行。这里就可以引出指令的概念了，一个小步骤就是一次指令操作，操作码就是指令码，两个 8 bit 的二进制数就是操作数。指令明确 ALU 要干什么，数据就是 ALU 要处理的数据，指令和数据合起来就是程序了。

### 指令集

程序就是由程序指令和程序数据构成的。下面把一次加法操作，比如，0b1 + 0b1 = 0b10，转换成程序。首先需要依次加载两个数字，然后需要知道是加法，然后需要进行运算，最后需要存储运算结果。

这里面有几种不同的操作：加载、加法、存储。这三个操作囊括了一次加法操作所需要的全部操作，构成了一个操作集合。实际的运行过程中需要给每个操作设计对应的电路来实现，这些不同的电路可以用指令码来进行标记。那么和操作一样，这些指令码也属于一个指令码集合，这就是指令集的概念。

### 控制单元

有了指令集之后，就可以设计电路来识别这些指令了。比如，0b0001 表示加载、0b0010 表示加法、0b0011 表示存储。每一个指令码对应一个电路（见图：**cpu.drawio.html 2-2**），当输入一个指令码的时候，整个电路只有和这个指令码匹配的那一处电路会导通（见图：**cpu.drawio.html 2-4**）。

识别出指令码是哪一个之后，就可以进行后续的操作了。比如，让电路连接内存，写入或者读取数据；或者让电路连接 ALU 进行运算并得到运算结果。这块的电路可以和 ALU、内存等组件独立开来成为单独一个模块，这就是控制单元。

### CPU

ALU 和控制单元就组成了中央处理器（Central Processing Unit、CPU）的核心。除了这两个玩意，CPU 里面还有寄存器和高速缓存（cache）等其他的组件。

上文的那个一次加法操作的整个程序是可以存储在内存中的，比如，内存篇里面提到的那块可以存储 256 个字节的内存。但是跑这个程序的时候，直接操作内存搞，搞起来会很复杂，所以会需要一些额外的空间临时存储一些数据，这些额外的空间通常由寄存器来实现。这里继续使用上文的一次加法操作来说明。

首先，构成一次加法操作的每一步指令是需要分析的。但是把内存线路直接接到控制单元上直接产生依赖显然不是一个好设计。可以从内存里把指令拿出来，暂存在控制单元里，然后再进行分析，这就是指令寄存器的概念。然后，程序跑到哪一步了也是需要记录的，这就是指令地址寄存器的概念。

两个操作数肯定是放在两个内存地址上的，这里肯定需要一个一个取出来，也就是需要两个能暂存操作数的地方。这里就是很普通的，用于存数据的寄存器，叫寄存器 A 和寄存器 B 好了。另外，ALU 计算的结果和标志位也需要处理。

最后，程序的每一次指令操作都需要电信号来触发，这个由时钟组件来控制，时钟会定时发射电脉冲。比如，每过一秒，就发射一次电脉冲，驱动整个系统运作一次。到这里，就可以画一个简单的结构图了，分为两部分：执行程序的 CPU 和存储程序的内存（见图：**cpu.drawio.html 2-6**）。

### CPU 执行程序的过程

#### 定义指令集和程序

这里假设这个 CPU 一次可以处理 8 bit 的数据，也就是一条指令是 8 bit。一条指令有包括指令码和操作数，这里假设前 4 bit 作为指令码，后 4 bit 作为操作数。

指令码属于指令集，这里假设指令集为：0b0001，从内存读取数据到寄存器 A；0b0010 从内存读取数据到寄存器 B；0b0011 把寄存器 A 和寄存器 B 的数据交给 ALU 做加法运算，然后把输出的结果放到寄存器 A，这个步骤不需要操作数；0b0100，把寄存器 A 的数据写入内存；

最后，假定指令的操作数都是内存地址。这样就可以把这个程序大概写出来了（见图：**cpu.drawio.html 2-8**）。假设两个数据都是 1，分别存储在内存地址 1100 和 1101 上。执行一次加法运算，然后把结果存储到内存地址 1110 上。

#### 执行程序的过程

把存有程序的内存和 CPU 连接起来（见图：**cpu.drawio.html 2-10-2**），假设指令地址寄存器从 0 开始，那么程序的执行过程如下：

1、将内存地址 0b0000 上的指令 0b00011100 加载到指令寄存器。分析指令的指令码，0b0001 表示从内存读取数据到寄存器 A。分析指令的操作数，0b1100 表示内存地址。于是去内存地址 0b1100 上，把数据 0b00000001 加载到寄存器 A。内存地址 0b0000 上的指令执行结束，指令地址寄存器 +1 得到 0b0001。（见图：**cpu.drawio.html 2-10-4-2、2-10-4-4、2-10-4-6**）

2、将内存地址 0b0001 上的指令 0b00101101 加载到指令寄存器。分析指令的指令码，0b0010 表示从内存读取数据到寄存器 B。分析指令的操作数，0b1101 表示内存地址。于是去内存地址 0b1101 上，把数据 0b00000001 加载到寄存器 B。内存地址 0b0001 上的指令执行结束，指令地址寄存器 +1 得到 0b0010。（见图：**cpu.drawio.html 2-10-6-2、2-10-6-4、2-10-6-6**）

3、将内存地址 0b0010 上的指令 0b00110000 加载到指令寄存器。分析指令的指令码，0b0011 表示把寄存器 A 和寄存器 B 的数据交给 ALU 做加法运算，然后把输出的结果放到寄存器 A，这个步骤不需要操作数。（见图：**cpu.drawio.html 2-10-8-2、2-10-8-4**）

于是先把寄存器 A 和寄存器 B 的数据交给 ALU 做加法运算，得到 0b00000010。因为这个时候寄存器 A 还连着 ALU 呢，直接把数据给过去，数据就又传给 ALU 了。所以控制单元会先暂存运算结果，然后切断寄存器 A 和 ALU 的连接，然后再把运算的结果放到寄存器 A 上。内存地址 0b0010 上的指令执行结束，指令地址寄存器 +1 得到 0b0011。（见图：**cpu.drawio.html 2-10-8-6、2-10-8-8**）

4、将内存地址 0b0011 上的指令 0b01001110 加载到指令寄存器。分析指令的指令码，0b0100 表示把寄存器 A 的数据写入内存。分析指令的操作数，0b1110 表示内存地址。于是去内存地址 0b1101 上，把数据 0b00000010 存进去。内存地址 0b0011 上的指令执行结束，指令地址寄存器 +1 得到 0b0100。（见图：**cpu.drawio.html 2-10-10-2、2-10-10-4、2-10-10-6**）

到这里，这个程序就已经执行结束了，后面的内存地址上如果存储了指令，那属于另外一个程序。

## 参考（reference）

- Crash Course Computer Science（计算机科学速成课）
  - [bilibili](https://www.bilibili.com/video/BV1EW411u7th)
  - [CrashCourse 字幕组](https://github.com/1c7/crash-course-computer-science-chinese)
  - [Youtube 原视频](https://www.youtube.com/playlist?list=PL8dPuuaLjXtNlUrzyH5r6jN9ulI)