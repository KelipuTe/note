---
draft: false
date: 2022-11-08 08:00:00 +0800
lastmod: 2022-11-08 08:00:00 +0800
title: "使用 Golang 实现 ORM 框架 -- INSERT、UPDATE、DELETE"
summary: "ORM 框架中使用 Builder 设计模式构建 MySQL 的 INSERT、UPDATE、DELETE 语句的过程、不同数据库 INSERT 语句不一样的问题的处理方式。"
toc: true

categories:
- framework(框架)

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- framework(框架)
- orm
- mysql
- sqlite3
- golang
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> go version go1.19 windows/amd64

### 资料

- [{orm-go}](https://github.com/KelipuTe/orm-go)/v20/
- <a href="/drawio/computer-science/programming-language/framework/orm/orm.drawio.html">orm.drawio.html</a>
- 注意：orm.drawio.html 里面的这个类图和流程图都是和 v20 版本的代码对应的最终形态。

### 前言

这篇是接着 SELECT 后面的。INSERT、UPDATE、DELETE 三个加起来都没有 SELECT 复杂。其中 INSERT 因为涉及到不同数据库的方言的处理，相对 UPDATE 和 DELETE 会更复杂一点。

本篇主要涉及：INSERT 语句的构造过程、不同数据库 INSERT 语句不一样的问题的处理方式、UPDATE 语句的构造过程、DELETE 语句的构造过程。

### 分析 INSERT 语句的使用场景

和 SELECT 的分析套路一样。

这里实现的是一个简单的 ORM，INSERT 的部分就先处理下面这几种。

- INSERT INTO 表(列) VALUES(值)
- INSERT INTO 表(列) VALUES(值1),(值2)
- INSERT INTO 表(列) VALUES(值) ON DUPLICATE KEY UPDATE 列=VALUES(值)
- INSERT INTO 表(列) VALUES(值) ON DUPLICATE KEY UPDATE 列=值

从上面的 SELECT 语句结构可以看出，SELECT 语句大致可以分成几个部分。

- INSERT INTO 表(列)
- VALUES(值)
- ON DUPLICATE KEY UPDATE (表达式)

然后再去看一下 MySQL 的官方文档中对 INSERT 语句的定义和说明。

### MySQL INSERT Statement

MySQL 8 官方文档中，关于 INSERT 语句的描述：

- [13.2.6 INSERT Statement](https://dev.mysql.com/doc/refman/8.0/en/insert.html)

把文档给出的结构和上面的那几种比较一下，提取出文档中相关的描述部分，大概是下面这部分。

```
INSERT
    [INTO] tbl_name
    [(col_name [, col_name] ...)]
    { {VALUES | VALUE} (value_list) [, (value_list)] ... }
    [ON DUPLICATE KEY UPDATE assignment_list]
```

### 构造语句

#### 从最简单的开始

先从最简单的 `SELECT INTO 表(列) VALUES(值)` 开始。

表和列很熟悉了，直接从元数据里搞。值也是一样的，构建元数据的时候，只通过反射获取了结构体属性的类型，值就是把结构体属性上的值也拿出来。然后按照把语句拼起来就可以了。INSERT 单个值的搞定了，多个值的循环单个值的步骤就行了。

#### 处理 ON CONFLICT

`ON CONFLICT` 在 MySQL 里面就是指的 `ON DUPLICATE KEY UPDATE` 也可以叫 `UPSERT`。这里单独拿出来说它是因为 ORM 框架一般不会只支持一种数据库。当需要支持多种数据库的时候，不同数据库的 SQL 语法之间冲突的部分就需要处理。这里用 MySQL 和 SQLite3 做演示。

在 MySQL 里面，语句是这样的：

- INSERT INTO 表(列) VALUES(值) ON DUPLICATE KEY UPDATE 列=VALUES(值)
- INSERT INTO 表(列) VALUES(值) ON DUPLICATE KEY UPDATE 列=值

SQLite 官方文档中，关于 INSERT 语句和 ON CONFLICT 的描述：

- [INSERT](https://sqlite.org/lang_insert.html)
- [The ON CONFLICT Clause](https://sqlite.org/lang_conflict.html)

在 SQLite3 里面，语句是这样的：

- INSERT INTO 表(列) VALUES(值) ON CONFLICT (列) DO UPDATE SET 列=excluded.值;
- INSERT INTO 表(列) VALUES(值) ON CONFLICT (列) DO UPDATE SET 列=值;

这两个数据库的 ON CONFLICT 语法结构是不一样的，所以在构造 SQL 语句的之后需要分开处理。这里需要对两个数据库的 ON CONFLICT 进行抽象。然后通过观察每个数据库的 SQL 语句，也就是 ON CONFLICT 后面的两种赋值方式，也会得到一个抽象。同样的，MySql 官方文档里也已经告诉你了，这个抽象叫 assignment_list（这里叫赋值表达式）。这个东西 UPDATE 语句里面也会用到，后面再说。

处理的时候先处理完 INSERT 前面一样的部分，然后再处理 ON CONFLICT 的部分。这个地方处理的时候，会有一个从处理公共部分的 INSERT 查询构造器跳到处理不同部分的 ON CONFLICT 查询构造器的步骤。这里可以用在 INSERT 查询构造器里面定义一个抽象的方式，然后构造 INSERT 查询构造器的时候把 ON CONFLICT 查询构造器传进来。另外一个思路是，INSERT 查询构造器处理完公共部分之后，把自己交给一个 ON CONFLICT 查询构造器。完了 ON CONFLICT 查询构造器处理完之后，在把他自己交给刚才的 INSERT 查询构造器，由 INSERT 查询构造器继续后面的执行 SQL 和处理结果集的步骤。

#### 关于方言抽象


