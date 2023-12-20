---
draft: false
date: 2023-02-28 08:00:00 +0800
title: "TTY"
summary: "TTY；"
toc: true

categories:
  - 计算机

tags:
  - 计算机
---

## 反向链接

[终端](/计算机/终端)；
[伪终端](/计算机/伪终端)；

## 资料

图：./TTY.drawio

## 正文

### TTY

TTY 也泛指计算机的终端设备，说 TTY 的时候可能是 tty 也可能是 pts，大部分情况下是 pts。

对于用户空间来说，这两个没有区别。

对于内核来说，tty 的另一端连接的是终端模拟器，pts 的另一端连接的是 ptmx，
终端模拟器和 ptmx 都只负责维护会话和转发数据。

终端模拟器的另一端连接的是键盘和显示器这样等硬件。
ptmx 的另一端连接的是用户空间的 xterm、sshd 等应用程序。

如果有多个终端连接上来的话，终端驱动会创建多个 TTY 与这些终端一一对应。
**见图：TTY.drawio**。

在 Linux 的终端窗口里使用 tty 命令可以查看当前 bash 关联到哪个 tty 或者 pts。

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

上面的 FD 这一列是文件描述符，0u、1u、2u 就是 
stdin(0u)、stdout(1u)、stderr(2u)，即标准输入、标准输出、标准错误输出。
