---
draft: false
title: "MySQL 索引"
summary: "MySQL 索引；"
toc: true

categories:
  - 数据库

tags:
  - 计算机
  - 数据库
  - mysql

date: 2022-05-26 08:00:00 +0800
---

## 正文

### 索引

索引（index）可以提高查询效率，就像各种类型的目录一样。

### 为什么用 B+ 树

可以用于提高读写效率的数据结构有很多。

- 等值查询：哈希表、数组、跳表、搜索树（平衡二叉树、红黑树、B 树、B+ 树）等。
- 范围查询：数组、双向链表、双向跳表、搜索树（B 树、B+ 树）等。

数据库的场景要求等值查询和范围查询都要有。

同时符合两个要求的有：数组、双向跳表、搜索树（B 树、B+ 树）等。
数组适用于静态存储的场景，插入和删除代价太大，用在这里不合适。

B+ 树相较于跳表：
- 跳表极端情况会退化为链表，平衡性差。
- 跳表需要消耗的内存更多。（除开数据，跳表指针更多）

B+ 树相较于二叉树：
- B+ 树的高度更低，意味着查询性能更好。
- 二叉树（平衡、红黑）的再平衡过程触发地更频繁。
- B+ 树叶子结点被串联起来了，更适合范围查询。
- B+ 树非叶子结点没有存数据，可以直接放内存里。

B+ 树相较于 B 树：
- B+ 树叶子结点被串联起来了，更适合范围查询。
- B+ 树非叶子结点没有存数据，可以直接放内存里。
- 插入和删除的时候，B 树可能需要移动结点。B 树因为结构要求，如果上层结点被删了，是一定要调整结点的。B+ 树可以不调整结点，上层结点只是索引用的可以不删除，只删除叶子结点。
- 在范围查询的时候，B 树中序遍历访问的结点数量比 B+ 树使用叶子结点的双向链表结构访问的结点数量多。这意味着 B 树可能需要更多的硬盘 IO。

核心就几个问题：
- 查询效率，数据库查询需要一个可预期的查询时间。
- 内存容量有限，数据长度无限，所以索引不能带数据。
- 需要尽量减少硬盘 IO，内存操作是很快的。

### 索引分类

角度不同分类结果就不同：

- 聚簇索引（主键索引）：索引的叶子结点存储的是数据行。
- 非聚簇索引（二级索引）：索引的叶子结点存储的是主键，需要回表。
- 主键索引：数据列不允许重复，不允许为 NULL，一个表只能有一个主键索引。
- 唯一索引：数据列不允许重复，允许为 NULL 值，一个表允许多个列创建唯一索引。
- 普通索引：数据列允许重复，允许为 NULL 值，一个表允许多个列创建普通索引。
- 组合索引（联合索引）：多个数据列组合在一起创建的索引。
- 前缀索引：索引只包含某列的前面一部分。比如，用 varchar64 前面的 32 个字符作索引。
- 全文索引：用于支持文本模糊查询的。
- 哈希索引：使用哈希算法的索引， InnoDB 引擎并不支持。

二级索引里同时有索引字段和主键字段。
所以相当于创建了一个联合索引 (普通字段,主键字段)。

创建联合索引，相当于创建多个索引。
比如，创建 (a,b,c) 联合索引，相当于创建 (a)、(a,b)、(a,b,c) 三个索引。
使用联合索引时，需要遵循最左匹配原则。

### 回表

如果某个查询语句使用了二级索引，但是查询的数据是整行记录。
这时在二级索引找到主键后，需要回到聚簇索引中获得整行记录。

这个过程叫作回表，整个查询过程需要查两棵索引树。
如果条件允许，尽量使用主键查询，这样就没有回表的步骤了。

### 索引覆盖

如果某个查询语句使用了二级索引，而且查询的数据就是主键或者索引字段。
这时在二级索引找到主键后，不需要回到聚簇索引中获得用户记录了，
因为二级索引中已经存储了主键的值和索引字段的值。
这个过程叫索引覆盖，整个查询过程只查了一棵索引树，它可以显著提升查询性能。

### 索引可能失效的场景

`!=` 或者 `like` 查询。

- `a != 1`，这种有索引也用不了，满足要求的数据太多了。
- `a like '%ab%'`，有索引用不了，因为要一条一条匹配。
- `a like 'ab%'`，这种能用上索引，类似前缀索引的逻辑。

a 列都是正数，`a > 0`，有索引和没索引都会拿全表的数据。

字段区分度不大时，用不用索引差别不大。
比如，只有 0 和 1 的 status 列。查 `a = 1` 和全表扫描没什么区别。

表中的数据太少时，用不用索引差别不大。

使用了特殊表达式。比如，数学运算、函数等。

另外，还可以用命令，强制要求数据库用索引。
不过一般用不到这些东西，逼不得已的时候再说。

- FORCE INDEX（强迫使用索引）
- USE INDEX（使用索引）
- IGNORE INDEX（忽略索引）

### 最左匹配原则

假设创建了 (a,b,c) 联合索引。

那么可以得到下面这三个结论：
- a 是有序的；索引 (a)。
- 在 a 确定的情况下 b 有序；索引 (a,b)。
- 在 a 和 b 都确定的情况下 c 有序；索引 (a,b,c)。

最左匹配原则就是查询时，从联合索引最左边的列开始筛选。

- `A = a1`，用索引 (a)。
- `A != a1`，用不了索引。
- `A = a1 AND B = b1`，用索引 (a,b)。
- `A = a1 AND B != b1`，用索引 (a)，然后在结果集里扫描 b。
- `A = a1 OR B = b1`，用索引 (a)，然后全表扫描 b，求并集。
- `A = a1 AND B > b1`。用索引 (a,b)。
- `A = a1 AND B > b1 AND C = c1`。用索引 (a,b)。

### 索引和 NULL

NULL 可以表达 "不知道"、"不存在"、"不合法" 的意思。

IS NULL 和 IS NOT NULL 都可以使用索引。

唯一索引允许有多行的值都是 NULL 的情况存在。

但是，大多数场景中，使用 NULL 都是一个比较差的实践。

### 索引的代价

- 索引本身需要存储，消耗磁盘空间。
- 运行时，索引需要先加载到内存，消耗内存空间。
- 增删改时，需要同步维护索引，消耗时间。

### 修改索引

修改索引的时候，数据量大的表和数据量小的表，实施方案是完全不一样的。

修改索引或者说表定义变更的核心问题是数据库会加表锁，直到修改完成。
所以当发现 MySQL 性能不行了，准备新加一个索引时。如果这个表的数据很多，那么在执行加索引的命令时，整张表可能都会被锁住几分钟甚至几个小时。

大表表结构变更是一件很麻烦的事情。一般可以考虑的方案有 3 种：
停机变更；业务低谷期变更；创建新表，然后将旧表的数据迁移过去。

-------------------------


### InnoDB

InnoDB 使用的索引模型是 B+ 树。在 InnoDB 中，每一个索引都对应一棵 B+ 树，

索引是存放在数据页（文件）里的，它要占据物理空间。B+
树为了维护索引有序性，在插入新值的时候会做必要的维护。如果页已经满了，这时候需要申请一个新的页，然后挪动部分数据过去，这个过程称为页分裂。当相邻的两个页由于删除了数据，利用率很低之后，又会将两个页做合并。

- 对主键字段建立的索引是聚簇索引，主键索引只有一个
- 聚簇索引的叶子结点存放的是实际数据，也就是用户记录
- 对普通字段建立的索引是二级索引，二级索引可以创建多个
- 二级索引的叶子结点存放的不是实际数据，而是主键值

对于主键索引，一般建议使用自增主键。因为自增主键长度小，而二级索引的叶子结点存放的是主键值，这样二级索引占的空间就小。

由于数据页的大小是固定的，索引字段长度越小，一个页内就可以放的越多，这样整体涉及的页就越少。涉及的页越少意味着硬盘 IO 越少。

对于二级索引，一般建议使用长度小、重合度小的字段，重合度越小的字段查询起来效率越高。



### change buffer

change buffer 用来更新非主键索引或唯一索引的二级索引 B+ 树的。它是一个可选项，默认是打开的。

change buffer 是 buffer pool 里的一块区域，存储的是最新的索引数据的变更。主要解决的是随机读硬盘的 IO 问题，将数据从硬盘读入内存涉及随机
IO，是数据库里面成本最高的操作之一。

当需要更新一个二级索引树的数据页时，如果页在内存中就直接更新，而如果页还没有在内存中的话，在不影响数据一致性的前提下，InnoDB
会将这些更新操作缓存在 change buffer 中，这样就不需要从磁盘中读入这个数据页了。在下次查询需要访问这个页的时候，将页从硬盘读入内存，然后执行
change buffer 中与这个页有关的操作。通过这种方式就能保证这个数据逻辑的正确性。

虽然名字叫作 change buffer，实际上它是可以持久化的数据，可以看成也是一个数据页。也就是说，change buffer
在内存中有拷贝，也会被写入到硬盘上。同时，change buffer 也是需要写 redo log 的。所以 redo log 里不仅有针对普通数据页的改动记录，也有
change buffer 的记录。

将 change buffer 中的操作应用到原数据页，得到最新结果的过程称为 merge。除了访问这个数据页会触发 merge 外，系统有后台线程会定期
merge。在数据库正常关闭（shutdown）的过程中，也会执行 merge 操作。显然，如果能够将更新操作先记录在 change
buffer，减少读硬盘，语句的执行速度会得到明显的提升。而且，数据读入内存是需要占用 buffer pool 的，所以这种方式还能够避免占用内存，提高内存利用率。

对于写多读少的业务来说，页面在写完以后马上被访问到的概率比较小，此时 change buffer
的使用效果最好。反过来，假设一个业务的更新模式是写入之后马上会做查询，那么即使满足了条件，将更新先记录在 change
buffer，但之后由于马上要访问这个数据页，会立即触发 merge 过程。这样随机访问 IO 的次数不会减少，反而增加了 change buffer
的维护代价。对于这种业务模式来说，change buffer 反而起到了副作用。

### change buffer 和 redo log

redo log 主要节省的是随机写磁盘的 IO 消耗（转成顺序写），而 change buffer 主要节省的则是随机读磁盘的 IO 消耗。

假设要执行一条语句：`mysql> insert into t(id,v) values(id1,v1),(id2,v2);`。假设当前 v 索引树的状态，查找到位置后，v1
所在的数据页在内存（InnoDB buffer pool）中，v2 所在的数据页不在内存中。下图是带 change buffer 的更新状态图。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/database/mysql/index_change_buffer_and_redo_log_insert.drawio.png">
</div>

更新语句做了如下的操作：

- page1 在内存中，直接更新内存。（图中 1）
- page2 没有在内存中，就在内存的 change buffer 区域，记录下"往 page2 插入一行"这个信息。（图中 2）
- 将上述两个动作记入 redo log 中。（图中 3、4）

做完上面这些，事务就可以完成了。

现在假设要执行一条语句：`select * from t where v in (v1,v2);`。下图是查询执行的流程图。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/database/mysql/index_change_buffer_and_redo_log_select.drawio.png">
</div>

- 读 page1 的时候，直接从内存返回。
- 读 page2 的时候，需要从硬盘读入内存中，然后应用 change buffer 里面的操作日志，生成一个正确的版本并返回结果。

### 唯一索引和普通索引的比较

这两类索引在查询能力上是没差别的，主要考虑的是对更新性能的影响。所以，建议尽量选择普通索引。

#### 查询过程

- 对于普通索引来说，查找到满足条件的第一个记录后，需要查找下一个记录，直到碰到第一个不满足条件的记录。
- 对于唯一索引来说，由于索引定义了唯一性，查找到第一个满足条件的记录后，就会停止继续检索。

查询过程，这两个没有明显的性能上的差距，普通索引需要读下一页这种属于少数情况。

#### 更新过程

对于目标数据在内存中的情况，两者都可以很快的完成更新操作，区别在于目标数据不在内存中的情况。

- 对于普通索引来说，因为没有额外的判断条件，直接将更新记录在 change buffer 中，语句执行就结束了。
- 对于唯一索引来说，是不能使用 change buffer 的，因为唯一索引需要保证数据不重复，所以一定要去硬盘中把数据对应的数据页读出来，然后判重。

所以针对更新操作密集的字段，需要慎重添加唯一索引。

### 索引下推

对于联合索引 (a,b,c)，SQL 语句 `where a=1 and c=3` 是能用到索引的，不过只能用到 a 的索引。这种属于索引截断。

MySQL 5.6 引入索引下推（index condition pushdown）优化，可以在索引遍历过程中，对索引（这里是 联合索引 (a,b,c)）中包含的字段（这里是
c）先做判断，直接过滤掉不满足条件的记录，减少回表次数。

如果用到了索引下推，那么 explain 的结果里的 extra 字段会显示 `Extra=Using index condition`。

### 索引失效（Index Invalid）

#### like

使用 `like '%xxx'` 或者 `like '%xxx%'` 的时候索引会失效。因为索引树是按照索引值有序排列的，只能进行前缀比较。

但是有个特殊的情况，如果查询的字段全部在索引树中，这个时候虽然索引失效了，但是依然会用到索引树，通过直接扫描整个索引树拿到需要的数据。因为聚簇索引树保存了全部的数据，它一定比二级索引树大，所以扫描二级索引树会快一点。

#### 联合索引非最左匹配

如果创建了一个联合索引 (a,b,c)。那么：

- `where a=1`、`where a=1 and b=2`、`where a=1 and b=2 and c=3` 都能用到索引。
- `where b=2`、`where c=3`、`where b=2 and c=3` 都用不到索引。
- 特别的 `where a=1 and c=3` 是索引截断，MySQL 5.6 及以后的版本会用到索引下推。

#### or

如果在 or 前的条件列是索引列，而在 or 后的条件列不是索引列，那么索引会失效。因为 or
的含义就是两个只要满足一个即可，因此只有一个条件列是索引列是没有意义的，只要有条件列不是索引列，就会进行全表扫描。

#### 对索引进行表达式计算

如果 id 字段有索引，那么 `where id + 1 = 10`
语句就用不到索引，因为索引保存的是索引字段的原始值，对索引进行表达式计算后的值不一定在索引里，只能全表扫描。但是 `where id = 10 - 1`
语句就可以用到索引。

#### 索引隐式类型转换

MySQL 在遇到字符串和数字比较的时候，会自动把字符串转为数字，然后再进行比较。`select '10' > 9; // 结果是 1` 通过这个例子就可以测试转换规则。

当索引字段是字符串的时候，如果 sql 语句是 `where 索引字段 = 123`，这时就会把索引字段转换成数字，也就不走索引了。

当索引字段是数字的时候，如果 sql 语句是 `where 索引字段 = '123'`，这时就会把 `'123'` 转换成数字，这个时候就走索引。

### 字符串字段加索引

有的字符串字段的值可能很长，直接加索引可能非常占空间。

有几种解决方案可以解决这个问题：

- 前缀索引：截取字符串最前面的几个字符作为索引。使用前缀索引时，如果定义好长度，就可以做到既节省空间，又不用额外增加太多的查询成本。但是覆盖索引就用不了了，因为不知道截取的字符串完不完整，是一定要回表的。
- 倒序存储：如果字符串前缀的区分度不大（比如身份证），就可以用倒序然后再取前缀。
- 哈希值字段：把要索引的字符串字段计算一个哈希值，放到一个哈希值字段上，然后在哈希值字段上建立索引，查询的时候使用字符串和哈希值一起查询。

## 参考

- [【Mysql核心剖析系列】Change Buffer 与 Redo Log的区别](http://www.icodebang.com/article/247207)
