---
draft: false
date: 2022-11-08 08:00:00 +0800
lastmod: 2022-11-08 08:00:00 +0800
title: "使用 Golang 实现 ORM 框架 -- INSERT"
summary: "INSERT 语句的构建过程。"

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

### MySQL INSERT Statement

MySQL 8 官方文档中，关于 INSERT 语句的描述：

- [13.2.6 INSERT Statement](https://dev.mysql.com/doc/refman/8.0/en/insert.html)

```
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [(col_name [, col_name] ...)]
    { {VALUES | VALUE} (value_list) [, (value_list)] ... }
    [AS row_alias[(col_alias [, col_alias] ...)]]
    [ON DUPLICATE KEY UPDATE assignment_list]

INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    SET assignment_list
    [AS row_alias[(col_alias [, col_alias] ...)]]
    [ON DUPLICATE KEY UPDATE assignment_list]

INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [(col_name [, col_name] ...)]
    { SELECT ... 
      | TABLE table_name 
      | VALUES row_constructor_list
    }
    [ON DUPLICATE KEY UPDATE assignment_list]

value:
    {expr | DEFAULT}

value_list:
    value [, value] ...

row_constructor_list:
    ROW(value_list)[, ROW(value_list)][, ...]

assignment:
    col_name = 
          value
        | [row_alias.]col_name
        | [tbl_name.]col_name
        | [row_alias.]col_alias

assignment_list:
    assignment [, assignment] ...
```

本人常用的大概是下面这部分，所以 ORM 框架也主要实现下面这部分。

```
INSERT
    [INTO] tbl_name
    [(col_name [, col_name] ...)]
    { {VALUES | VALUE} (value_list) [, (value_list)] ... }
    [ON DUPLICATE KEY UPDATE assignment_list]
```
