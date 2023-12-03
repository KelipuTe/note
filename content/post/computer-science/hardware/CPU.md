---
draft: false
date: 2023-02-21 08:00:00 +0800
title: "CPU"
summary: "CPU；"
toc: true

categories:
  - 硬件

tags:
  - 计算机科学
  - 硬件
---

## 正文

图：/content/post/computer-science/hardware/CPU.drawio

词汇表：

- 中央处理器、central processing unit、CPU
- 可编程的、programmable
- 时钟、clock
- 时钟频率、clock rate
- 时钟周期、clock cycle
- 赫兹、hertz、Hz
- 超频、over clocking
- 降频、under clocking
- 动态频率调整、dynamic frequency scaling
- 机器周期、machine cycle
- 指令周期、instruction cycle
- 指令平均时钟周期数、cycles per instruction、CPI
- 指令流水线、instruction pipeline
- 平行、parallelize
- 动态排序、dynamically reorder
- 依赖、dependencies
- 乱序执行、out-of-order execution
- 分支预测、branch prediction
- 推测执行、speculative execution
- 超标量处理器、superscalar processor
- 多核处理器、multi-core processor
- 多个独立 CPU、multiple independent CPU

### CPU

CPU 是整个计算机的核心。它的核心部分由运算器和控制器组成。

[运算器](/post/computer-science/hardware/运算器)

[控制器](/post/computer-science/hardware/控制器)

CPU 的强大之处在于，如果输入不同的指令，就会执行不同的动作。
它是可编程的，这意味着它可以执行不同的程序。

[指令](/post/computer-science/program/指令)

[程序](/post/computer-science/program/程序)

除了核心的运算器和控制器之外还有寄存器、CPU 高速缓存、总线等。

[寄存器](/post/computer-science/hardware/寄存器)

[CPU_高速缓存](/post/computer-science/hardware/CPU_高速缓存)

### 时钟

控制器里面有个重要的部件叫时钟。时钟负责管理 CPU 的工作节奏。

时钟以精确的时间间隔触发脉冲信号，控制单元根据这个信号推进 CPU 的内部操作。
因为电的传输需要时间，所以时钟的节奏不能太快。

时钟频率，单位是赫兹，10 Hz 代表时钟 1 秒触发脉冲信号 10 次。
时钟周期又叫振荡周期，每一次脉冲信号高低电平的转换就是一个周期，时钟频率的倒数。
时钟频率越高，时钟周期越短，CPU 工作速度越快。

超频可以提升 CPU 的工作效率，但是会产生散热问题或因为跟不上时钟频率产生乱码。
降频可以省电，CPU 不需要时刻保持全速工作。现代计算机都有动态频率调整的功能。

### 一条指令的执行过程

执行一条指令一般分为三个步骤：

- 取指令（instruction fetch、IF）
- 指令译码（instruction decode、ID）
- 执行指令（execute、EX）

CPU 会根据指令周期的不同阶段区分取指令还是取操作数。

初始化时，所有的寄存器都初始化为 0。
RAM 中存入一段程序，假设地址从 0 开始。

取指令阶段，负责拿到指令。

CPU 将指令地址寄存器连接到 RAM，寄存器的值为 0，因此 RAM 返回地址 0 上的数据。
返回的数据会被复制到指令寄存器里。

指令译码阶段，搞清楚指令要干什么。

CPU 用控制单元判断指令寄存器里的指令的操作码是哪一个。
这里假设译码的结果为："操作码：读取内存数据放入寄存器 A；地址：9"。

执行阶段，执行译码的结果。

从 RAM 地址 9 的位置取出操作数，放到寄存器 A 里。
执行完成后，关闭所有电路，指令地址寄存器 +1，本次指令流程结束。

### CPU 执行程序的过程

[CPU_执行程序的过程](/post/computer-science/hardware/CPU_执行程序的过程)

### 机器周期和指令周期

机器周期，又叫 CPU 周期，是 CPU 完成一个基本操作所需的时间。

指令周期是 CPU 执行一条指令所需的时间。

一条指令的执行过程划分为若干个阶段，每一阶段完成一个基本操作。

程序的机器周期数 = 程序的指令数 x 指令平均时钟周期数

程序的 CPU 执行时间 = 程序的机器周期数 x 时钟周期

### 指令流水线

指令流水线是指把一条指令的操作分成多个小步骤，每个步骤由专门的电路完成。
然后，再使用平行处理的方式提高指令的执行效率。

高级 CPU 会动态排序有依赖关系的指令。
然后乱序执行，这里的乱序是和原来的顺序相比较的。

高级 CPU 会使用分支预测技术，提前把指令放入流水线。
另外还有推测执行技术。

### 其他提升性能的方法

超标量处理器：一个时钟周期完成多个指令。

多核处理器：一个 CPU 有多个内核，可同时处理多个指令流。

多个独立 CPU：多个独立 CPU 同时工作。
