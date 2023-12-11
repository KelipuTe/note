---
draft: true
date: 2022-05-24 08:00:00 +0800
lastmod: 2022-05-24 08:00:00 +0800
title: "存储引擎（Storage Engine）"
summary: "存储引擎（Storage Engine）"
toc: true

categories:
- database

tags:
- computer-science(计算机科学)
- database(数据库)
- mysql
---

### MyISAM

锁粒度为表锁，加锁开销小，但是并发小。

MyISAM 则没有事务日志

MyISAM 可以直接定位到数据所在的内存地址，可以直接找到数据。

MyISAM 索引是非聚簇索引。MyISAM 索引的叶子节点存储的是行数据地址，需要再寻址一次才能得到数据。

表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概
率最高，并发度最低。

### InnoDB

锁粒度为行锁，加锁开销更大，但是支持更大的并发。

行锁可以解决脏独和不可重复读的问题。

行锁容易出现死锁。

InnoDB 是有事务日志，数据库崩溃后可以恢复

InnoDB 在查询过程中，是需要维护数据缓存，而且查询过程是先定位到行所在的数据块，然后在从数据块中定位到要查找的行;

InnoDB 索引是聚簇索引，InnoDB 的主键索引的叶子节点存储着行数据，因此主键索引非常高效。

InnoDB 非主键索引的叶子节点存储的是主键和其他带索引的列数据，因此
查询时做到覆盖索引会非常高效。

行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概
率最低，并发度也最高。

### 参考

- [极客时间](https://time.geekbang.org/)
  - [MySQL 实战 45 讲](https://time.geekbang.org/column/intro/100020801?tab=catalog)