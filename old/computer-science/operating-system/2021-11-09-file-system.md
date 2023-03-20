---
title: "file system（文件系统）"
create_date: 2021-11-09 08:00:00 +0800
date: 2021-11-09 08:00:00 +0800
tags: computer-science operating-system
comment: false
show_author_profile: true
show_subscribe: false
---

### 文件

file（文件）的本质都是二进制编码，不同的 file format（文件类型），编码格式不同。

在正确读取数据之前，需要知道这些数据是什么。关于数据的数据叫 meta data（元数据）。元数据存在文件开头，在实际数据前面，因此也叫 header（文件头）。

### 目录文件

directory file（目录文件）记录其他文件的信息。比如：file name（文件名）、file extension（扩展名）、元数据、created time（创建时间）、last modified（最后修改时间）、owner（所有者）、can be read or written（读写权限）等。最重要的是目录文件还会记录文件的 begin storage（起始位置）和 file length（文件长度）。如果修改了文件，同时也要更新目录文件。

目录文件经常在存储器的 location 0（最开头），方便查找。

### 文件系统

file system（文件系统）专门管理文件。文件系统让我们不必在关心文件实际存储位置，让整理和访问文件更加方便。

移动文件只需要修改目录文件中的数据。不需要移动实际存储空间的数据。

删除文件时只是把目录文件中对应的数据删掉了，并没有修改实际存储空间的数据。原先放置被删除文件数据的空间块会被标记为可用块。在这些空间块被新的数据覆盖之前，被删除文件的数据还在原处。

常见的结构有 flat file system（平面文件系统）和 hierarchical file system（分层文件系统）。

#### 平面文件系统

所有文件都在同一个层次的叫平面文件系统。平面层次系统有一个问题，文件都是首尾相连的，如果前面的文件增加了数据，会导致后面的文件被覆盖。

现代计算机为了解决这个问题，会把空间划分成 block（块），这样就会有一些 extra space（预留空间）方便改动。目录文件记录文件的位置的方式变成记录文件所在空间块的编号，拆分文件，存在多个块里。

#### 分层文件系统

分层文件系统就是现在常用的文件夹套文件夹的形式。最顶层的目录文件也叫 root directiry（根目录），所有其它文件和文件夹都在根目录下。每个文件夹里都有一个自己的目录文件。目录文件里还需要额外的元数据来区分目录和文件。

#### 碎片整理

如果不进行 defragmentation（碎片整理），大量的文件会以 fragmentation（碎片）的形式存储在各个地方。而磁盘巡道是需要时间的，所以这种情况会影响读取速度。现代计算机会把数据来回移动，排列成正确的顺序，方便读取。

### 参考

- Crash Course Computer Science（计算机科学速成课）
- [bilibili](https://www.bilibili.com/video/BV1EW411u7th)
- [CrashCourse 字幕组](https://github.com/1c7/crash-course-computer-science-chinese)
- [Youtube 原视频](https://www.youtube.com/playlist?list=PL8dPuuaLjXtNlUrzyH5r6jN9ulI)