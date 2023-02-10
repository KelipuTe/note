---
draft: false
date: 2021-12-06 08:00:00 +0800
lastmod: 2022-02-09 08:00:00 +0800
title: "Linux 文档"
summary: "man 命令；linux 在线文档；"
toc: true

categories:
- operating-system(操作系统)

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- linux-c
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> VMware Workstation Pro 16<br/>
> Ubuntu 22.04<br/>
> gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0

搞 linux c 编程，一定要学会看 linux 的文档，遇到问题，仔细看看文档，一般都能解决。

### man

可以在 linux 系统中使用 `man` 命令或者使用在线文档（[man7.org](https://man7.org/index.html) 或者 [Linux man pages online](https://man7.org/linux/man-pages/index.html)）查看 linux 相关文档。

关于 man 命令具体怎么用可以看：[man(1) - an interface to the system reference manuals](https://man7.org/linux/man-pages/man1/man.1.html)。

#### 在 centos 的 docker 容器中无法使用 man 命令

因为 centos 的 docker 的镜像，它将一些东西精简了，所以不能够使用一些常用的命令，比如 man 等等。

需要在 `/etc/yum.conf` 配置文件里面修改一下，注释掉：`tsflags=nodocs`，这个配置禁用了一些软件包。注释完成之后重新安装 man 即可使用。

```
yum -y install man
yum -y install man-pages
```

### Linux man pages online

在线文档从 [man7.org](https://man7.org/index.html) 或者 [Linux man pages online](https://man7.org/linux/man-pages/index.html) 都能进的去。

`man7.org` 页面点击 `Online manual pages` 可以进到 `Linux man pages online` 页面。`Linux man pages online` 页面点击 `by section` 可以进到列表页面，在列表页面比较好找。
