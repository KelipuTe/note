---
draft: false
date: 2021-12-06 08:00:00 +0800
lastmod: 2023-02-10 08:00:00 +0800
title: "搞 Linux C 编程的注意点"
summary: "学会看 linux 文档；注意代码运行的目标环境；"
toc: true

categories:
- operating-system(操作系统)

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- linux-c
---

### 学会看 linux 文档

搞 linux c 编程，一定要学会看 linux 文档。遇到问题，仔细看看文档，一般都能解决。

#### man

在 linux 系统中，可以使用 `man` 命令查看 linux 文档。

关于 man 命令具体怎么用可以看：[man(1) - an interface to the system reference manuals](https://man7.org/linux/man-pages/man1/man.1.html)。

#### Linux man pages online

在线文档从 [man7.org](https://man7.org/index.html) 或者 [Linux man pages online](https://man7.org/linux/man-pages/index.html) 都能进的去。

`man7.org` 页面点击 `Online manual pages` 可以进到 `Linux man pages online` 页面。

`Linux man pages online` 页面点击 `by section` 可以进到列表页面，在列表页面比较好找。

#### 在 centos 的 docker 容器中无法使用 man 命令

因为 centos 的 docker 的镜像，它把一些东西精简了，所以有一些命令不能使用。比如，man。

需要修改 `/etc/yum.conf` 配置文件。注释掉 `tsflags=nodocs` 这一行。这个配置禁用了一些软件包。修改配置文件后，重新安装 man，即可使用。

```
yum -y install man
yum -y install man-pages
```

#### 笔记中出现的文档

- [fork(2) - create a child process](https://man7.org/linux/man-pages/man2/fork.2.html)
- [vfork(2) - create a child process and block parent](https://man7.org/linux/man-pages/man2/vfork.2.html)
- [errno(3) - number of last error](https://man7.org/linux/man-pages/man3/errno.3.html)
- [getpid(2) - get process identification](https://man7.org/linux/man-pages/man2/getpid.2.html)
- [getppid(2) - get process identification](https://man7.org/linux/man-pages/man2/getppid.2.html)

### 注意代码运行的目标环境

编程的时候，要注意代码运行的目标环境。同一段代码，在 Ubuntu 和 Centos 上运行的时候，整体的系统调用过程应该是差不多的，但是细节可能不一样。

举个例子，在 Ubuntu 22.04 和 Centos 7 上分别跑 helloworld。加载 libc.so.6 文件的这个步骤，在Ubuntu 22.04 上加载的是 `/lib/x86_64-linux-gnu/libc.so.6`，在 Centos 7 上加载的是 `/lib64/libc.so.6`。
