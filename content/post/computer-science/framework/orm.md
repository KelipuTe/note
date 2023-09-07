---
draft: false
date: 2023-09-06 08:00:00 +0800
title: "实现一个简单的 orm 框架"
summary: "构造 select 语句；builder 设计模式；元数据；结果集；
构造 insert 语句；构造 update 语句；构造 delete 语句；"
toc: true

categories:
  - framework(框架)

tags:
  - computer-science(计算机科学)
  - framework(框架)
---

## 前言

这里实现的是一个简单的 orm 框架，主要目的是研究原理和设计。
编程语言用的 golang，数据库用的 mysql 和 sqlite。

orm 框架的核心功能主要有两个：
- 1、把数据结构转换成 sql 语句。
- 2、处理 sql 语句的执行结果。
- 其他的功能，都建立在这两个功能的基础上。

数据结构转换成 sql 语句这没啥好说的，就是把 golang 的数据结构转换成对应的 sql 语句。
处理 sql 语句的执行结果主要指的是，自动把 select 语句执行的结果装到对应的数据结构里去。

这两个功能的实现过程都会用到反射操作和内存操作，需要先有这两个方面的知识。

实践的环境：
- CPU AMD64(x86_64)
- Windows 11 家庭版
- go version go1.19 windows/amd64

## 资料

## 正文

### 构造 select 语句

#### 分析使用场景

这里就处理下面这几种：
- select ... from ...
- select ... from ... where ...
- select ... from ... group by ... having ...
- select ... from ... order by ...
- select ... from ... limit ... offset ...
- join 查询和子查询

从上面的 SELECT 语句结构可以看出，SELECT 语句大致可以分成几个部分。

- SELECT {*|列|聚合函数} as 别名
- FROM {表|JOIN 查询|子查询} as 别名
- WHERE 条件（普通条件，与或非，where in）
- GROUP BY 列 HAVING 条件
- ORDER BY 列,asc|desc
- LIMIT n OFFSET m

这样大概的内容和处理流程就有了。就是构建 SELECT 语句需要哪些东西，构建的大概流程是什么样的。然后再去看一下 MySQL 的官方文档中对 SELECT 语句的定义和说明。

### 元数据

## 参考
