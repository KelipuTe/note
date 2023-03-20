---
draft: false
date: 2022-05-24 08:00:00 +0800
lastmod: 2022-05-24 08:00:00 +0800
title: "数据库范式（Database Normalization）"
summary: "数据库范式（Database Normalization）"
toc: true

categories:
- database

tags:
- computer-science(计算机科学)
- database(数据库)
---

### 几个概念

- 候选码：在关系中，能唯一标识一条记录的⼀组属性
- 主码（主键）：从候选码中选定一个成为主码
- 主属性：候选码中的属性都是主属性

### 1NF（1 Normal form、第一范式）

关系（数据表）中的所有的属性（字段）都是不可拆分的。

### 2NF（2 Normal form、第二范式）

在 1NF 基础上，消除非主属性对主码的部分依赖。非主属性完全依赖于主码，而不是依赖于主码的一部分。

### 3NF（3 Normal form、第三范式）

在 2NF 基础上，消除非主属性对主码的传递依赖。非主属性只依赖于主码，而不是可以依赖于其他非主属性。

### BCNF（Boyce-Codd Normal Form、巴斯-科德范式）

在 3NF 基础上，消除主属性对主码的部分依赖和传递依赖。
