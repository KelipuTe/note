---
draft: false
date: 2022-02-09 08:00:00 +0800
lastmod: 2022-02-09 08:00:00 +0800
title: "hosts 文件"
summary: "hosts 文件"
toc: true

categories:
- windows

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- windows
---

hosts 文件所在的目录：`C:\Windows\System32\drivers\etc`。

hosts 文件是一个没有扩展名的系统文件，用于储存计算机网络中各节点信息，负责将主机名称映射到相应的 IP 地址。简单地说它的作用就是用来存储一些常用的网络域名和与其对应的 IP 地址。

hosts 文件通常用于补充或取代网络中 DNS 的功能。和 DNS 不同的是，计算机的用户可以直接对 hosts 文件进行控制。

当用户输入一个需要登录的网址时，系统就会先去 hosts 文件中查找。如果找到了就立即打开该网址，如果找不到就去 DNS 域名解析服务器中查找。
