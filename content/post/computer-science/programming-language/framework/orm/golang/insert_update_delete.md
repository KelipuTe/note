---
draft: false
date: 2022-11-14 08:00:00 +0800
lastmod: 2022-11-14 08:00:00 +0800
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

assignment:
    col_name = 
          value
        | [tbl_name.]col_name

assignment_list:
    assignment [, assignment] ...
```

### 构造 INSERT 语句

#### 从最简单的开始

先从最简单的 `INSERT INTO 表(列) VALUES(值)` 开始。

表和列很熟悉了，直接从元数据里搞。值也是一样的，构建元数据的时候，只通过反射获取了结构体属性的类型，值就是把结构体属性上的值也拿出来。然后按照把语句拼起来就可以了。INSERT 单个值的搞定了，多个值的循环单个值的步骤就行了。

#### 提取查询构造器

在构造 INSERT 语句的时候会发现，在构造 SELECT 语句的时候使用到的一些构造的方法，在构造 INSERT 语句的时候也会用的到。所以这里可以提取一个抽象出来，就叫它 SQL 查询构造器好了。这样后面 UPDATE 和 DELETE 语句构造的时候也可以用的上，不用重复的去写。创建 SELECT 查询构造器或者 INSERT 查询构造器这些具体的构造器的时候，组合一个 SQL 查询构造器进去就行了。

#### 处理 ON CONFLICT

`ON CONFLICT` 在 MySQL 里面就是指的 `ON DUPLICATE KEY UPDATE` 也可以叫 `UPSERT`。这里单独拿出来说它是因为 ORM 框架一般不会只支持一种数据库。当需要支持多种数据库的时候，不同数据库的 SQL 语法之间冲突的部分就需要处理。这里用 MySQL 和 SQLite3 做演示。

在 MySQL 里面，语句是这样的：

- INSERT INTO 表(列) VALUES(值) ON DUPLICATE KEY UPDATE 列=VALUES(列)
- INSERT INTO 表(列) VALUES(值) ON DUPLICATE KEY UPDATE 列=值

SQLite 官方文档中，关于 INSERT 语句和 ON CONFLICT 的描述：

- [INSERT](https://sqlite.org/lang_insert.html)
- [The ON CONFLICT Clause](https://sqlite.org/lang_conflict.html)

在 SQLite3 里面，语句是这样的：

- INSERT INTO 表(列) VALUES(值) ON CONFLICT (列) DO UPDATE SET 列=excluded.列;
- INSERT INTO 表(列) VALUES(值) ON CONFLICT (列) DO UPDATE SET 列=值;

这两个数据库的 ON CONFLICT 语法结构是不一样的，所以在构造 SQL 语句的之后需要分开处理。这里需要对两个数据库的 ON CONFLICT 进行抽象。然后通过观察每个数据库的 SQL 语句，也就是 ON CONFLICT 后面的两种赋值方式，也会得到一个抽象。同样的，MySql 官方文档里也已经告诉你了，这个抽象叫 assignment_list（这里叫赋值表达式）。这个东西 UPDATE 语句里面也会用到，后面再说。

处理的时候先处理完 INSERT 前面一样的部分，然后再处理 ON CONFLICT 的部分。这个地方处理的时候，会有一个从处理公共部分的 INSERT 查询构造器跳到处理不同部分的 ON CONFLICT 查询构造器的步骤。这里可以用在 INSERT 查询构造器里面定义一个抽象的方式，然后构造 INSERT 查询构造器的时候把 ON CONFLICT 查询构造器传进来。

另外一个思路是，INSERT 查询构造器处理完公共部分之后，把自己交给一个 ON CONFLICT 查询构造器。完了 ON CONFLICT 查询构造器处理完之后，在把他自己交给刚才的 INSERT 查询构造器，由 INSERT 查询构造器继续后面的执行 SQL 和处理结果集的步骤。

#### 关于方言抽象的注入

从上面的过程可以发现，方言抽象是需要注入到 INSERT 查询构造器里面的。这就意味着每次创建新的 INSERT 查询构造器的时候，都需要根据到底是生成哪个数据库的 SQL 语句进而注入对应的方言抽象的实例。这显然是非常麻烦的，有一个更好的位置可以放这个方言抽象，这个位置就是数据库抽象。

仔细想一下，不管是哪种具体的查询构造器，构造出来的语句最终都是要靠数据库实例去执行的。而且方言本身也是因为需要使用不同的数据库从而产生的设计，所以把方言放到数据库抽象里面是最合适的。查询构造器在构造 SQL 的时候，就可以直接调用数据库实例里面的方言实例去处理不同数据库里面冲突的部分。

#### INSERT 语句就差不多了

### UPDATE

UPDATE 就比较简单了。同样的，把文档放在这里，然后把相关的描述部分提取出来。

- [13.2.15 UPDATE Statement](https://dev.mysql.com/doc/refman/8.0/en/update.html)

```
UPDATE table_reference
    SET assignment_list
    [WHERE where_condition]
```

table_reference 和 where_condition 在 SELECT 那里处理过，assignment_list 在 INSERT 那里处理过。

### 注意这个 assignment（赋值语句）

assignment 接口有两种实现。一个是列，对应 `列=VALUES(列)` 这种语句。另一种是常规的赋值语句，对应 `列=值` 这种的。这里注意一下 `列=VALUES(列)` 这种的，这种语句在 INSERT 的不同的数据库方言里和 UPDATE 语句里面，它的实现又是各不相同的。目前的实现是先把 assignment 接口断言成列，然后不同的数据库方言和 UPDATE 自己实现自己的逻辑。

### DELETE

DELETE 更简单。同样的，把文档放在这里，然后把相关的描述部分提取出来。

- [13.2.2 DELETE Statement](https://dev.mysql.com/doc/refman/8.0/en/delete.html)

```
DELETE FROM tbl_name
    [WHERE where_condition]
```

table_reference 在 SELECT 那里处理过，assignment_list 在 INSERT 那里处理过。

### 执行语句

这三个语句的执行和 SELECT 不一样，它们是没有处理结果集一说的，都是返回影响了多少行。顶多 INSERT 有一个返回插入的行的 id 的功能。所以这里需要再写一套和数据库交互的逻辑，因为返回值不一样。如果是在 Golang 里面，执行 SQL 调用的方法就是不一样的，SELECT 用的是 query() 这三个用的是 exec()。

### 全流程结束

到这里，生成 INSERT、UPDATE、DELETE 语句、执行语句、处理返回值，就都处理完了。同样的，设计上的东西，看类图、流程图会更加直观。细节上的实现，直接看代码，这个靠说是说不太清楚的。
