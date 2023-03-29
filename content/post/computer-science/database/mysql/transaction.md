---
draft: false
date: 2022-05-25 08:00:00 +0800
lastmod: 2022-05-25 08:00:00 +0800
title: "事务（Transaction）"
summary: "事务（Transaction）"
toc: true

categories:
- database

tags:
- computer-science(计算机科学)
- database(数据库)
- mysql
---

### ACID

- 原子性（Atomicity）
- 一致性（Consistency）
- 隔离性（Isolation）
- 持久性（Durability）

### 隔离性

#### 隔离级别

当 MySQL 上有多个事务同时执行的时候，就可能出现脏读（dirty read）、不可重复读（non-repeatable read）、幻读（phantom read）的问题，为了解决这些问题，就有了隔离级别的概念。

隔离得越严实，效率就会越低，不同的隔离级别要在隔离与效率之间寻找一个平衡点。

SQL 标准的事务隔离级别包括：读未提交（read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（serializable）。

- 读未提交：一个事务还没提交时，它做的变更就能被别的事务看到。
- 读提交：一个事务提交之后，它做的变更才会被其他事务看到。
- 可重复读：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。
- 串行化：对于同一行记录，后访问的事务必须等前一个事务执行完成，才能继续执行。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/database/mysql/transaction_isolation.drawio.png">
</div>

对于上图的事物 A，在不同的隔离级别下 v1、v2、v3 的值会不同。

- 读未提交：v1 = 2；v2 = 2；v3 = 2。事务 B 的更新在提交前就能被 A 看到。
- 读提交：v1 = 1；v2 = 2；v3 = 2。事务 B 的更新在提交后才能被 A 看到。
- 可重复读：v1 = 1；v2 = 1；v3 = 2。事务在执行期间看到的数据前后必须是一致的。
- 串行化：v1 = 1；v2 = 1；v3 = 2。在事务 B 执行 `把 1 改成 2` 的时候，会被锁住。直到事务 A 提交后，事务 B 才可以继续执行。

在实现上，MySQL 会创建一个事务视图（一致性读视图、一致性视图、consistent read view、read view），用于支持 读提交（Read Committed、RC）和 可重复读（Repeatable Read、RR）隔离级别的实现。访问数据的时候以视图的逻辑结果为准。

- 在读未提交下，直接返回记录上的最新值，没有视图概念
- 在读提交下，这个视图是在每个 SQL 语句开始执行的时候创建的
- 在可重复读下，这个视图是在事务启动时创建的，整个事务存在期间都用这个视图
- 在串行化下，直接用加锁的方式来避免并行访问。

### 多版本并发控制

在 MySQL 中，每条记录在更新的时候都会同时记录一条回滚操作。记录上的最新值，通过回滚操作，都可以得到前一个状态的值。

在可重复读下，假设一个值从 1 被按顺序改成了 2、3、4，在 undo log（回滚日志）里面就会有类似下面的记录。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/database/mysql/transaction_undo_log.drawio.png">
</div>

当前值是 4，但是在查询这条记录的时候，不同时刻启动的事务会有不同的 read-view。如图中看到的，在视图 A、B、C 里面，这一个记录的值分别是 1、2、4，同一条记录在系统中可以存在多个版本，就是数据库的多版本并发控制（MVCC）。对于 read-view A 要得到 1，就必须将当前值依次执行图中所有的回滚操作得到。

回滚日志不会一直保留，当系统判断没有事务再需要用到这些回滚日志时，也就是当系统里没有比这个回滚日志更早的 read-view 的时候，回滚日志会被删除。

因此尽量不要使用长事务。长事务意味着系统里面会存在很老的事务视图。由于这些事务随时可能访问数据库里面的任何数据，所以这个事务提交之前，数据库里面它可能用到的回滚记录都必须保留，这就会导致大量占用存储空间。除了对回滚段的影响，长事务还占用锁资源，也可能拖垮整个库。

### 一致性视图

在可重复读下，事务在启动的时候就拍了个快照。注意，这个快照是基于整库的。快照不需要拷贝整个库的数据，它是基于事务 ID 实现的。

InnoDB 里面每个事务有一个唯一的事务 ID（transaction id）。它是在事务开始的时候向 InnoDB 的事务系统申请的，是按申请顺序严格递增的。

每行数据也都是有多个版本的。每次事务更新数据的时候，都会生成一个新的数据版本，并且把事务 ID 赋值给这个新的数据版本（记为 row trx_id）。同时，旧的数据版本要保留，并且在新的数据版本中，能够有信息可以直接拿到它。也就是说，数据表中的一行记录，其实可能有多个版本（row），每个版本有自己的 row trx_id。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/database/mysql/transaction_transaction_id.drawio.png">
</div>

上图是同一行数据的 4 个版本，当前最新版本是 v4，a 的值是 4，它被事务 ID 为 25 的事务更新的，因此它的 row trx_id 也是 25。图中的 u1、u2、u3 就是回滚日志。

按照可重复读的定义，一个事务启动的时候，能够看到所有已经提交的事务结果。但是之后，这个事务执行期间，其他事务的更新对它不可见。

因此，一个事务只需要以启动的时刻为准，如果一个数据版本是在启动之前生成的，就认；如果是在启动以后才生成的，就不认，必须要找到它的上一个版本。如果上一个版本也不可见，就得继续往前找。还有，如果是这个事务自己更新的数据，它自己还是要认的。

在实现上，InnoDB 为每个事务构造了一个数组，用来保存这个事务启动瞬间，当前正在活跃的所有事务 ID。活跃指的就是，启动了但还没提交。

数组里面事务 ID 的最小值记为低水位，当前系统里面已经创建过的事务 ID 的最大值加 1 记为高水位。这个视图数组和高水位，就组成了当前事务的一致性视图。

而数据版本的可见性规则，就是基于数据的 row trx_id 和这个一致性视图的对比结果得到的。这个视图数组把所有的 row trx_id 分成了几种不同的情况。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/database/mysql/transaction_row_trx_id.drawio.png">
</div>

对于当前事务的启动瞬间来说，一个数据版本的 row trx_id，有以下几种可能：

- 如果落在绿色部分，表示这个版本是已提交的事务或者是当前事务自己生成的，数据可见
- 如果落在黄色部分，包括两种情况
  - 若 row trx_id 在数组中，表示这个版本是由还没提交的事务生成的，不可见
  - 若 row trx_id 不在数组中，表示这个版本是已经提交了的事务生成的，可见
  - 落在黄色部分表示 $低水位 < row trx\_id < 高水位$，但是 row trx_id 不一定在视图数组中。
- 如果落在红色部分，表示这个版本是由将来启动的事务生成的，数据不可见

以上就是 InnoDB 如何利用所有数据都有多个版本的这个特性，实现了秒级创建快照的能力。

### 一个一致性视图的例子

需要注意事务的启动时机。`begin/start transaction` 命令并不是一个事务的起点，在执行到命令之后的第一个操作 InnoDB 表的语句，事务才真正启动。如果你想要马上启动一个事务，可以使用 `start transaction with consistent snapshot` 这个命令。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/database/mysql/transaction_read_view_demo.drawio.png">
</div>

这里做如下假设：

- 事务 A 开始前，系统里面只有一个活跃事务 ID 是 99
- 事务 A、B、C 的版本号分别是 100、101、102，且当前系统里只有这四个事务
- 三个事务开始前，`id=1,a=1` 这一行数据的 row trx_id 是 90。

#### 事务 A

跟事务 A 查询逻辑有关的操作如下图。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/database/mysql/transaction_read_view_demo_session_a.drawio.png">
</div>

事务 A 查询语句的读数据流程是这样的：

- 事务 A 的视图数组是 `[99,100]`
- 找到 a=3，row trx_id=101，比高水位大，处于红色区域，不可见
- 找到上一个历史版本1，row trx_id=102，比高水位大，处于红色区域，不可见
- 找到上一个历史版本2，row trx_id=90，比低水位小，处于绿色区域，可见
- 最终事务 A 的查询语句输出 a=1。

#### 事务 B

事务 B 是更新逻辑，流程和事务 A 的查询逻辑不一样。

跟事务 B 更新逻辑有关的操作如下图。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/database/mysql/transaction_read_view_demo_session_b.drawio.png">
</div>

这里需要用到当前读（current read）规则：更新数据都是先读后写的，而且只能读当前的值。除了 update 语句外，select 语句如果加锁（lock in share mode 或 for update），也是当前读。

因此，在更新的时候，当前读拿到的数据是 a=2，更新后生成了新版本的数据 a=3，这个新版本的 row trx_id 是 101。

在事务 B 执行查询语句的时候，事务 ID 是 101，数据版本也是 101，说明是自己的更新，可以直接使用，所以查询得到 a=3。

### 参考

- [极客时间](https://time.geekbang.org/)
  - [MySQL 实战 45 讲](https://time.geekbang.org/column/intro/100020801?tab=catalog)
    - 03 | 事务隔离：为什么你改了我还看不见？
    - 08 | 事务到底是隔离的还是不隔离的？