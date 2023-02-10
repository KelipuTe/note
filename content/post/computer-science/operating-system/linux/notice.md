---
draft: false
date: 2021-12-06 08:00:00 +0800
lastmod: 2023-02-10 08:00:00 +0800
title: "Linux C 编程的注意点"
summary: ""
toc: true

categories:
- operating-system(操作系统)

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- linux-c
---

其实就一个注意点，编程的时候，注意环境。同一段代码，在 Ubuntu 和 Centos 上运行的时候，整体的系统调用过程应该是差不多的，但是细节可能不一样。

举个例子，在 Ubuntu 22.04 和 Centos 7 上分别跑 helloworld。加载 libc.so.6 文件的这个步骤，在Ubuntu 22.04 上加载的是 `/lib/x86_64-linux-gnu/libc.so.6`，在 Centos 7 上加载的是 `/lib64/libc.so.6`。
