---
draft: false
date: 2022-06-07 08:00:00 +0800
lastmod: 2022-06-07 08:00:00 +0800
title: "在 VMware 虚拟机中安装 Linux"
summary: "安装 Ubuntu 22.04；"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- vmware
- vmware-workstation
- linux
- ubuntu
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> VMware Workstation Pro 16<br/>
> Ubuntu 22.04

### 安装 Ubuntu 22.04

#### 下载页面在哪

官网的下载路径：

- [官网](https://ubuntu.com/)
- 官网 --> 最上面的导航栏 --> 点击 "Download" --> 选择 "Ubuntu Desktop" --> 然后页面就会变成 "Download Ubuntu Desktop" 的页面，一般都是一个 LTS 版本，如果没有特别需求，直接点旁边的 "Download" 按钮就行了。
- 如果有特别的需求，在 "Download" 按钮的下面有一个 "see our alternative downloads" 链接，点击进入 [Alternative downloads](https://cn.ubuntu.com/download/alternative-downloads) 页面。
- 如果 "Alternative downloads" 页面还不能满足需求，在 "Alternative downloads" 页面的最下面，有一个 "Past releases and other flavours" 板块 --> 板块里面有一个 [Past releases](https://releases.ubuntu.com/)，可以在里面找需要的版本。

中文官网的下载路径：

- [中文官网](https://cn.ubuntu.com/)
- 官网 --> 最上面的导航栏 --> 点击 "下载" --> 选择 "桌面" --> 然后页面就会变成 "下载Ubuntu桌面系统" 的页面，一般都是一个 LTS 版本，如果没有特别需求，直接点旁边的 "下载" 按钮就行了。
- 如果有特别的需求，在 "下载" 按钮的下面有一个 "其他下载" 链接，点击进入 [其他下载](https://cn.ubuntu.com/download/alternative-downloads) 页面。
- 如果 "其他下载" 页面还不能满足需求，在 "其他下载" 页面的最下面，有一个 "历史版本和其他风味版" 板块 --> 板块里面有一个 [查看历史版本](https://releases.ubuntu.com/)，可以在里面找需要的版本。

#### 下载和安装

在刚才的下载页面，下载 Ubuntu 22.04 LTS 的安装包。

新建虚拟机：

- 打开 VMware Workstation Pro。
- 最上面的导航栏 --> 点击 "文件" --> 点击 "新建虚拟机" --> 一般一路下一步就行，中间需要选择上面下载的 Ubuntu 的安装包作为虚拟机的操作系统。
- 新建虚拟机完成后，启动虚拟机。

第一次进入操作系统，会弹出 Install（正常安装） 页面：

- Install --> 语言选择 English(US)（英语） --> 点击 Continue。
- Updates and other software --> 选择 Normal installation（正常安装） --> 其他的默认 --> 点击 Continue。
- Installation type --> 选择 Erase disk and install Ubuntu（清除硬盘并安装） --> 点击 Install Now --> 点击 Continue。
- Where are you? --> 选择 Shanghai --> 点击 Continue。
- Who are you? --> 填：计算机名称、用户名、密码 --> 选择 Log in automatically（自动登录） --> 点击 Continue。
- 其余直接点击下一步，然后等待安装完成。

### 共享文件夹

- 最上面的导航栏 --> 点击 "虚拟机" --> 点击 "设置"。
- 最上面的选项卡 --> 点击 "选项" --> 在下面的列表里找到 "共享文件夹" --> 点击 "共享文件夹"。
- 右边的 "文件夹共享" 里选择 "总是启用" --> 右边的 "文件夹" 里点击 "添加"。
- 点 "下一步" --> 命名共享文件夹 --> 主机路径就是本机的目录，名称就是映射到 Ubuntu 里的目录。

启动虚拟机，共享文件夹应该会被映射到 `/mnt/hgfs/上面的名称对应的目录`。如果没有 hgfs 目录就创建一个。如果看不到共享文件夹，就执行下面两条命令，然后再次进入 hgfs 目录就能看见了。

```
> vmware-hgfsclient
> sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
```

上面这样的操作是一次性的，下次开机又要重复操作。如果想让系统每次开机的时候自动挂载，需要编辑 `/etc/fstab`。这样系统启动时会自动将文件挂载到指定位置。

`sudo vim /etc/fstab`，使用管理员权限打开 `/etc/fstab`。在最后添加一行：`.host:/ /mnt/hgfs fuse.vmhgfs-fuse allow_other 0 0`。这样下次系统启动时就会自动挂载了。

### 常用软件

#### gcc

```
# 这个命令将会安装一系列软件包，包括 gcc、g++、make
# 一路 y 就行
> sudo apt install build-essential

# 可以通过输出 gcc 版本来检查是否安装成功
> gcc --version
gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0
```

#### git

```
> sudo apt-get update

# 安装依赖
> sudo apt-get install libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev
# 安装 git
> apt-get install git

# 可以通过输出 git 版本来检查是否安装成功
> git --version
```

#### netstat

```
> sudo apt-get install net-tools
```

#### vim

```
# 一路 y 就行
> sudo apt-get install vim-gtk
```

### reference（参考）

- [Download Ubuntu Desktop](https://ubuntu.com/download/desktop)
- [VMware虚拟机安装Ubuntu详解](https://zhuanlan.zhihu.com/p/477725832)
- [VMware虚拟机Ubuntu共享文件夹](https://blog.csdn.net/qq_16763983/article/details/121086240)
- [vmware ubuntu /mnt/hgfs 没有权限查看 找不到共享文件夹](http://dljz.nicethemes.cn/news/show-340389.html)
- [如何在 Ubuntu 20.04 上安装 GCC(build-essential)](https://blog.csdn.net/chiunbill/article/details/121803697)
- [Ubuntu上 git的安装与使用]((https://www.bilibili.com/read/cv7431710))
