---
draft: true
date: 2022-10-10 08:00:00 +0800
lastmod: 2022-10-10 08:00:00 +0800
title: "使用 Golang 实现 ORM 框架 -- SELECT"
summary: "使用 Builder 设计模式构建 MySQL 的 SELECT 语句的过程。"

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

### MySQL SELECT Statement

MySQL 8 官方文档中，关于 SELECT 语句的描述：

- [13.2.10 SELECT Statement](https://dev.mysql.com/doc/refman/8.0/en/select.html)

```
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
    [HIGH_PRIORITY]
    [STRAIGHT_JOIN]
    [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
    [SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr] ...
    [into_option]
    [FROM table_references
      [PARTITION partition_list]]
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}, ... [WITH ROLLUP]]
    [HAVING where_condition]
    [WINDOW window_name AS (window_spec)
        [, window_name AS (window_spec)] ...]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [into_option]
    [FOR {UPDATE | SHARE}
        [OF tbl_name [, tbl_name] ...]
        [NOWAIT | SKIP LOCKED]
      | LOCK IN SHARE MODE]
    [into_option]

into_option: {
    INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        export_options
  | INTO DUMPFILE 'file_name'
  | INTO var_name [, var_name] ...
}
```

本人常用的大概是下面这部分，所以 ORM 框架也主要实现下面这部分。

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

#### alias（别名）

- 列的别名：

A select_expr can be given an alias using AS alias_name.

- 表的别名：

A table reference can be aliased using tbl_name AS alias_name or tbl_name alias_name.

For each table specified, you can optionally specify an alias.

```
tbl_name [[AS] alias]
```

#### aggregate（聚合函数）

- [12.20 Aggregate Functions](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions-and-modifiers.html)
