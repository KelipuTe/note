---
draft: false
date: 2022-10-10 08:00:00 +0800
lastmod: 2022-10-10 08:00:00 +0800
title: "使用 Golang 实现简单的 ORM 框架 -- SELECT"
summary: "使用 Builder 设计模式构建 MySQL 的 SELECT 语句的过程。"
toc: true

categories:
- framework

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- framework
- orm
- mysql
- golang
---

### 前言

注意，这里实现的是一个简单的 ORM 框架，并不是一个完备的 ORM 框架，主要目的是研究原理和设计。

ORM 框架的核心功能主要有两个：1、把数据结构转换成 SQL 语句。2、处理 SQL 语句的执行结果。

数据结构转换成 SQL 语句这没啥好说的，在这个简单的 ORM 框架里面就是把 Golang 的结构体转换成 SQL 语句。

处理 SQL 语句的执行结果主要指的是，简化手动操作，自动把 SELECT 语句执行的结果装到对应的结构体里去。

另外，这两个功能的实现过程都会用到反射操作和内存操作，需要先有这两个方面的知识。

本篇主要涉及：SQL 语句的分析和抽象、SELECT 语句的分析和抽象、元数据的构造、结果集的处理。

### 分析 SELECT 语句的使用场景

这里实现的是一个简单的 ORM，SELECT 的部分就先处理下面这几种。

- SELECT 列 FROM 表 WHERE 条件
- SELECT 列 FROM 别名 WHERE 条件
- SELECT 聚合函数 FROM 表 WHERE 条件
- SELECT 列 AS 别名 FROM 表 AS 别名 WHERE 条件
- SELECT 列 FROM 表 WHERE 条件 LIMIT 数字 OFFSET 数字
- SELECT 列 FROM 表 WHERE 条件 ORDER BY 列
- SELECT 列 FROM 表 WHERE 条件 GROUP BY 列 HAVING 条件
- SELECT 列 FROM JOIN AS 别名 WHERE 条件
- SELECT 列 FROM 子查询 AS 别名 WHERE 条件
- SELECT 列 FROM 表 WHERE 列 IN 子查询取一列

从上面的 SELECT 语句结构可以看出，SELECT 语句大致可以分成几个部分。

- SELECT {列|聚合函数} AS 别名
- FROM {表|JOIN|子查询} AS 别名
- WHERE 条件
- GROUP BY 列 HAVING 条件
- ORDER BY 列
- LIMIT 数字 OFFSET 数字

这样大概的内容和处理流程就有了。就是构建 SELECT 语句需要哪些东西，构建的大概流程是什么样的。

然后再去看一下 MySQL 的官方文档中对 SELECT 语句的定义和说明。

### MySQL SELECT Statement

MySQL 8 官方文档中，关于 SELECT 语句的描述：

- [13.2.10 SELECT Statement](https://dev.mysql.com/doc/refman/8.0/en/select.html)

把文档给出的结构和上面的那几种比较一下，提取出文档中相关的描述部分，大概是下面这部分。

```
SELECT
    [DISTINCT]
    select_expr [, select_expr] ...
    [FROM table_references]
    [WHERE where_condition]
    [GROUP BY {col_name}, ...]
    [HAVING where_condition]
    [ORDER BY {col_name} [ASC | DESC], ...]
    [LIMIT {row_count | row_count OFFSET offset}]
    [FOR UPDATE]
}
```

还需要关注以下关于别名的部分：

- 列的别名：

> A select_expr can be given an alias using AS alias_name.

- 表的别名：

> A table reference can be aliased using tbl_name AS alias_name or tbl_name alias_name.<br/>
> For each table specified, you can optionally specify an alias.

```
tbl_name [[AS] alias]
```

还需要关注一下 JOIN 的部分：

- [13.2.11.2 JOIN Clause](https://dev.mysql.com/doc/refman/8.0/en/join.html)

主要是下面这部分，描述的就是 JOIN 的结构。

```
table_reference: {
    table_factor
  | joined_table
}

table_factor: {
    tbl_name [[AS] alias]
  | table_subquery [AS] alias [(col_list)]
  | ( table_references )
}

joined_table: {
    table_reference {JOIN} table_factor [join_specification]
  | table_reference {LEFT|RIGHT} JOIN table_reference join_specification
  | table_reference NATURAL [INNER | {LEFT|RIGHT} [OUTER]] JOIN table_factor
}

join_specification: {
    ON search_condition
  | USING (join_column_list)
}
```

也就是说 JOIN 的结构大概可以描述成下面这几种，而且还可以套娃。

- 表 JOIN 表
- （表 JOIN 表） JOIN 表
- 表 JOIN 子查询
- 子查询 JOIN 子查询

还需要关注一下子查询的部分：

- [13.2.13 Subqueries](https://dev.mysql.com/doc/refman/8.0/en/subqueries.html)

简单的理解，子查询就是构建了一个临时的表，只不过子查询的表的列不是固定的，而是动态生成的。

到这里 SELECT 语句的构成分析的就差不多了，下面是对 SELECT 语句的构成进行合理的抽象。

### 分析和抽象

其实抽象的工作是比较容易的，官方文档中的描述已经把抽象给出来了。

在这个简单的 ORM 框架里面，大概是下面这几个。

- select_expr 对应列和聚合函数
- table_references 对应表、JOIN、子查询
- where_condition 对应查询条件
- col_name 对应列

### 构造开始

先从最简单的 `SELECT 列 FROM 表` 开始。

这里就会遇到一个问题，列和表怎么来，这就涉及到元数据的问题。

未完。。。
