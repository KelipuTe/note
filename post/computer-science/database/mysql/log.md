---
draft: true
date: 2022-05-24 08:00:00 +0800
lastmod: 2022-05-24 08:00:00 +0800
title: "日志（Log）"
summary: "日志（Log）"
toc: true

categories:
- database

tags:
- computer-science(计算机科学)
- database(数据库)
- mysql
---

WAL（Write-Ahead Logging、写前日志）的关键点是先写日志，再写硬盘。

### redo log

redo log（重做日志） 是 InnoDB 引擎特有的日志。

redo log 是物理日志，记录的是在某个数据页上做了什么修改，比如某个字段值从 1 改成 2。

InnoDB 的 redo log 是一个固定大小环形结构，有点像循环链表。write pos 指针记录当前的位置，一边写一边后移。checkpoint 指针记录当前要擦除的位置，一边擦除一边后移。

当 write pos 指针绕一圈追到 checkpoint 指针的时候，就表示 redo log 满了。redo log 满了的时候就必须停下来，更新一点记录到硬盘里面，然后擦除一些记录，把 checkpoint 指针推进一下。

这里用 Update 语句为例。当有执行 Update 语句更新一条记录的时候，InnoDB 会先在内存中的数据页中更新这条记录，然后把这条记录写到 redo log 里面，到这里，这个更新操作就算完成了。InnoDB 会在适当的时候（系统比较空闲、redo log 满了），将这条记录更新到硬盘里面并清理 redo log。

这里的数据页是文件，redo log 也是文件。但是因为数据页是要找，是随机 IO，redo log 是追加，是顺序 IO，所以 redo log 效率要更高。

有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录也不会丢失，这个能力称为 crash-safe。

### binlog

binlog（归档日志）是 server 层的日志，所有引擎都可以使用。

binlog 是逻辑日志，记录的是语句的原始逻辑，比如某个字段值 +1。

binlog 是追加写入的，而且 binlog 文件写到一定大小后会自动切换到下一个。

binlog 没有 crash-safe 的能力，只能用于归档。

#### 写入格式

- statement：保存每一条会修改数据的 SQL，保存的时候会保存 SQL 执行的上下文信息。日志量少，节约 IO。
- row：保存哪条记录被修改，不需要保存上下文。但是有些操作会导致大量改动，日志量太大。
- mixed，一种折中的方案，普通操作使用 statement，无法使用 statement 时，使用 row。

### 二阶段提交

两阶段提交是为了让 redo log 和 binlog 之间的逻辑一致。

这里用 Update 语句为例：

- 1、在内存中的数据页中更新这条记录
- 2、写 redo log 
- 3、redo log 改成 prepare 状态
- 4、写 binlog
- 5、redo log 改成 commit 状态

崩溃分析：

- 3~4 崩了：redo log 发现没有 commit，回滚。
- 4~5 崩了：redo log prepare 而且 binlog 完整，自动提交

### 参考

- [极客时间](https://time.geekbang.org/)
  - [MySQL 实战 45 讲](https://time.geekbang.org/column/intro/100020801?tab=catalog)
    - 02 | 日志系统：一条SQL更新语句是如何执行的？