---
draft: false
date: 2023-09-06 08:00:00 +0800
title: "ORM 框架"
summary: "ORM 框架相关笔记的导航；"
toc: true

categories:
  - framework(框架)

tags:
  - computer-science(计算机科学)
  - framework(框架)
  - orm
---

## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- go version go1.20 windows/amd64

## 资料

- [{demo-golang}](https://github.com/KelipuTe/demo-golang)/orm/
- <a href="/drawio/computer-science/programming-language/framework/orm/orm.drawio.html">orm.drawio.html</a>

## 正文

这里实现的是一个简单的 ORM 框架，主要目的是研究原理和设计。
编程语言用 Golang，数据库用 Mysql 和 Sqlite。

ORM 框架的核心功能主要有两个：

- 1、把数据结构转换成 SQL 语句（把 Golang 的数据结构转换成对应的 SQL 语句）。
- 2、处理 SQL 语句的执行结果（把 select 语句执行的结果装到对应的数据结构里去）。
- 其他的功能，都建立在这两个功能的基础之上。

ORM 框架的核心功能一般基于反射操作实现，当反射操作的性能不够时，可以考虑使用内存操作进行优化。

[Go_反射](/计算机/programming-language/golang/Go_反射)

[ORM框架_查](/计算机/framework/ORM框架_查)

[ORM框架_增改删](/计算机/framework/ORM框架_增改删)

[ORM框架_中间件](/计算机/framework/ORM框架_中间件)

## 参考
