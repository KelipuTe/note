---
draft: false
date: 2023-02-28 08:00:00 +0800
lastmod: 2023-02-28 08:00:00 +0800
title: "终端和控制台"
summary: "终端；控制台；物理终端；软件仿真终端；伪终端；TTY；"
toc: true

categories:
- computer-science(计算机科学)

tags:
- computer-science(计算机科学)
- linux
---
## 前言

交叉笔记：[init 进程、进程间关系、作业、会话、守护进程](/post/computer-science/operating-system/linux/process02)

## 资料

- <a href="/drawio/computer-science/terminal_console.drawio.html">terminal_console.drawio.html</a>

## 正文

### 终端

计算机终端（computer terminal）是与计算机相连的一种输入输出设备。

早期有一种叫电传打字机的设备，它是一台独立于计算机的机器，通过物理线缆与计算机连接，可以完成计算机的输入输出功能。电传打字机的英文是 Teletype 或 Teletypewriter，缩写是 TTY，所以 TTY 也泛指计算机的终端设备。后来，电传打字机被键盘和显示器取代。

但是不管是电传打字机还是键盘和显示器，都是作为计算机的终端设备存在的。电传打字机属于物理终端。现在物理终端基本上已经不用了。图形界面出现之前，键盘和显示器的组合属于软件仿真终端，也就是软件模拟出来的终端。图形界面出现之后，键盘和显示器的组合演变成了伪终端，但是软件仿真终端还是可以用的。

### 控制台

这里以数控机床为例。数控机床可能有一个独立出来一整块，然后上面有很多按钮的地方，这些按钮可以直接控制机床上的设备。如果这地方是直接在机床表面的，就叫控制面板。如果这地方是被一个柜子锁起来的，就叫控制箱。如果这地方看上去像个台子，就叫控制台（console）。

控制台和终端其实不是一个东西。控制台是计算机本身就有的设备，而终端不是计算机本身就有的设备。控制台应该只有一个，但是终端可以有多个。计算机的控制台理论上应该是电源开关和电源灯那些玩意，电源开关是可以直接控制计算机的。终端是通过接口接上去的，终端和计算机本身的运行没有关系，它就是个额外的输入输出设备。

这里用 ubuntu 系统和 windows 系统为例。计算机刚启动的时候，是可以通过键盘直接控制计算机的，而且会输出内核的信息到显示器上。这个时候的键盘和显示器就类似于控制台。

当计算机启动完成，进入操作系统的桌面后，这时是可以开很多的命令行窗口的。这些命令行窗口里面不会输出内核的信息，而且可以各自独立的接收命令输入，然后执行程序并返回输出。这个时候的键盘和显示器就是终端。

不过，现在的控制台和终端都由硬件概念逐渐演化成了软件的概念。简单的说，第一个启动的，会输出内核的信息的那个终端就称为控制台，其他的终端就是终端。

### 物理终端

这里用电传打字机为例，画了基本的软硬件结构和交互。见图：**terminal_console.drawio.html 2-2**。

- UART（通用异步接收器和发射器）和 UART driver（UART 驱动）的功能简单的理解就是，将电传打字机的信号转换为计算机可以识别的信号。
- line discipline（行规范） 的功能简单的理解就是，提供一个编辑缓冲区和一些基本的命令。比如：回车命令把缓冲区的数据发送给 TTY 驱动；退格命令清除一个字符。
- TTY driver（终端驱动）的功能有两个：进行会话管理；处理终端设备的输入输出。

### 软件仿真终端

这里用键盘和显示器（注意这里是直连）为例，画了基本的软硬件结构和交互。见图：**terminal_console.drawio.html 20-2**。

差别在于用键盘驱动、显示器驱动、终端模拟器替换了 UART 和 UART 驱动。基本流程不变，只是输入输出从一个设备变成两个设备了。

在 linux 系统中，软件仿真终端在文件系统中的表示就是 /dev/tty1 ~ /dev/tty6。见图：**terminal_console.drawio.html 20-4**。

在 ubuntu 系统中，在图形界面的桌面按下 "CTRL" + "ALT" + "F1" ~ "F6" 就可以分别进入 tty1 ~ tty6 对应的软件仿真终端。软件仿真终端的界面是操作系统内核直接提供的，这种叫虚拟控制台（virtual console）。

按下 "CTRL" + "ALT" + "F7" 或者重新启动系统，就可以回到图形界面。在图形界面也可以打开终端，这种叫终端窗口（terminal window），属于伪终端。终端窗口就是常用的那个控制台黑窗口。

这两中终端在功能上其实没啥区别，如果哪天操作系统的图形界面炸掉了，但是别的地方没问题，那倒是可以切到软件仿真终端去救火。

### 伪终端

物理终端和软件模拟终端都需要直接连接外部设备，这两种方式对物理距离都是有限制的，伪终端（pseudo terminal，pty）的出现解决了这个问题。伪终端就是由终端模拟器提供的虚拟终端，终端模拟器是一种运行在用户空间的应用程序。

这里用 xterm 终端模拟器为例，画了基本的软件结构和交互。见图：**terminal_console.drawio.html 40-2**。

伪终端在操作系统内核中分为两部分：分别是不在 TTY 驱动中 PTY master side 和在 TTY 驱动中的 PTY slave side。master side 是接近用户的一端；slave side 是在虚拟终端上运行的 CLI（Command Line Interface，命令行接口）程序。

连接 master side 和 slave side 的是伪终端驱动。见图：**terminal_console.drawio.html 40-4**。

伪终端驱动会把写入 master side 的数据转发给 slave side 作为程序的输入。程序输出时会写入 slave side。伪终端驱动会把写入 slave side 的数据转发给 master side 作为可以读取的数据。

在 linux 系统中，当创建一个伪终端时。会调用 posix_openpt() 请求 ptmx 创建一个 pts。ptmx 就是 "/dev/ptmx"，对应 master side。创建出来的 pts 在 "/dev/pts/" 目录下，对应 slave side。

#### ssh

类似 PuTTY 这类通过 SSH 的方式远程连接到 linux 系统的终端模拟程序。如果从 linux 系统这边看的话，连接上来之后产生的用户进程，就是图 **terminal_console.drawio.html 40-4** 中 xterm process 的位置，它和 master side 交互。

建立连接的过程：

- 客户端（PuTTY）请求和 sshd 建立连接。
- sshd 验证通过后，会创建一个新的会话（session）。
- sshd 调用 posix_openpt() 请求 ptmx 创建一个 pts。如果创建成功，sshd 会得到一个连接到 ptmx 的文件描述符（fd）和一个 pts。ptmx 会自动维护这个 fd 和 pts 的关系。
- sshd 会把创建的会话和这个 fd（连接到 ptmx 的文件描述符）关联起来。同时 sshd 会创建 bash 进程，将 pts 和 bash 进程关联起来。

用 lsof 命令可以查看 "/dev/ptmx" 被哪些进程打开了。如果 sshd 请求 ptmx 创建一个 pts 成功。那么输出里应该会有像下面这样 'COMMAND 为 sshd，NAME 为 "/dev/ptmx' 的行。

```
> lsof /dev/ptmx
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    1191  dev    8u   CHR    5,2      0t0 6531 /dev/ptmx
...
```

收发消息的过程：

- 客户端（PuTTY）收到键盘的输入，通过 ssh 协议将数据发往 sshd。
- sshd 收到客户端的数据后，从自己管理的会话里找到该客户端对应的 fd，然后将数据写入 fd。
- ptmx 收到 sshd 的数据后，从自己管理的关系里找到该 fd 对应的 pts，然后将数据写入 pts。
- pts 收到 ptmx 的数据后，检查关联的前端进程组，将数据转发给前端进程组的组长进程。因为上面是创建 bash 进程和 pts 关联起来的，所以这里就是 bash 收到了数据。
- bash 收到 pts 的数据后，会对输入的数据进行处理，然后输出处理结果，输出的处理结果会被写入 pts。
- pts 收到 bash 的数据后，将数据写入 ptmx。
- ptmx 收到 pts 的数据后，从自己管理的关系里找到该 pts 对应的 fd，然后将数据写入 fd。
- sshd 收到 fd 的数据后，从自己管理的会话里找到该 fd 对应的客户端，然后将数据发往客户端。

见图：**terminal_console.drawio.html 40-6**。

#### 键盘和显示器

图形界面出现之后，键盘和显示器的组合演变成了伪终端，整个结构和 ssh 的差不多。只不过没有远程连接了，图形客户端需要把 ssh 客户端的功能也实现，然后直接和 master side 交互。见图：**terminal_console.drawio.html 40-8**。

### TTY

TTY 也泛指计算机的终端设备，说 TTY 的时候可能是 tty 也可能是 pts，大部分情况下是 pts。

对于用户空间来说，这两个没有区别。对于内核来说，tty 的另一端连接的是终端模拟器，pts 的另一端连接的是 ptmx，终端模拟器和 ptmx 都只负责维护会话和转发数据。终端模拟器的另一端连接的是键盘和显示器这样等硬件。ptmx 的另一端连接的是用户空间的 xterm、sshd 等应用程序。

如果有多个终端连接上来的话，终端驱动会创建多个 TTY 与这些终端一一对应。见图：**terminal_console.drawio.html 12-2**。

在 linux 的终端窗口里使用 tty 命令可以查看当前 bash 关联到哪个 tty 或者 pts。

```
> tty
/dev/pts/0
```

然后用 lsof 命令可以查看这个 tty 或者 pts 被哪些进程打开了。

```
> lsof /dev/pts/0
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash    2150  qqq    0u   CHR  136,0      0t0    3 /dev/pts/0
bash    2150  qqq    1u   CHR  136,0      0t0    3 /dev/pts/0
bash    2150  qqq    2u   CHR  136,0      0t0    3 /dev/pts/0
bash    2150  qqq  255u   CHR  136,0      0t0    3 /dev/pts/0
lsof    2163  qqq    0u   CHR  136,0      0t0    3 /dev/pts/0
lsof    2163  qqq    1u   CHR  136,0      0t0    3 /dev/pts/0
lsof    2163  qqq    2u   CHR  136,0      0t0    3 /dev/pts/0
```

上面的 FD 这一列是文件描述符，0u、1u、2u 就是 stdin(0u)、stdout(1u)、stderr(2u)，即标准输入、标准输出、标准错误输出。

## 参考（reference）

- [Linux 终端(TTY)](https://www.cnblogs.com/sparkdev/p/11460821.html)
- [Linux 伪终端(pty)](https://www.cnblogs.com/sparkdev/archive/2019/09/29/11605804.html)
- [Linux TTY/PTS概述](https://segmentfault.com/a/1190000009082089)
- [终端、虚拟终端、shell、控制台、tty的区别](https://blog.csdn.net/ltx06/article/details/52170852)
