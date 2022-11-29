---
draft: false
date: 2022-10-10 08:00:00 +0800
lastmod: 2022-10-10 08:00:00 +0800
title: "使用 Golang 实现简单的 ORM 框架 -- SELECT"
summary: "ORM 框架中使用 Builder 设计模式构建 MySQL 的 SELECT 语句的过程、元数据的构造、结果集的处理。"
toc: true

categories:
- framework(框架)

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- framework(框架)
- orm
- mysql
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

注意，这里实现的是一个简单的 ORM 框架，并不是一个完备的 ORM 框架，主要目的是研究原理和设计。

ORM 框架的核心功能主要有两个：1、把数据结构转换成 SQL 语句。2、处理 SQL 语句的执行结果。

数据结构转换成 SQL 语句这没啥好说的，在这个简单的 ORM 框架里面就是把 Golang 的结构体转换成对应的 SQL 语句。

处理 SQL 语句的执行结果主要指的是，简化手动操作，自动把 SELECT 语句执行的结果装到对应的结构体里去。

另外，这两个功能的实现过程都会用到反射操作和内存操作，需要先有这两个方面的知识。

本篇主要涉及：SELECT 语句的分析和抽象、SELECT 语句的构造过程、元数据的构造、结果集的处理。

### 分析 SELECT 语句的使用场景

这里实现的是一个简单的 ORM，SELECT 的部分就先处理下面这几种。

- SELECT ... FROM ...
- SELECT ... FROM ... WHERE ...
- SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ...
- SELECT ... FROM ... WHERE ... ORDER BY ...
- SELECT ... FROM ... WHERE ... LIMIT ... OFFSET ...
- SELECT ... AS ... FROM ... AS ... WHERE ...
- SELECT ... FROM JOIN
- SELECT ... FROM 子查询
- SELECT ... FROM ... WHERE ... IN ...

从上面的 SELECT 语句结构可以看出，SELECT 语句大致可以分成几个部分。

- SELECT {*|列|聚合函数} AS 别名
- FROM {表|JOIN|子查询} AS 别名
- WHERE 条件
- GROUP BY 列 HAVING 条件
- ORDER BY 列
- LIMIT 数字 OFFSET 数字

这样大概的内容和处理流程就有了。就是构建 SELECT 语句需要哪些东西，构建的大概流程是什么样的。然后再去看一下 MySQL 的官方文档中对 SELECT 语句的定义和说明。

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

主要是下面这部分，描述的是 JOIN 的结构。

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

也就是说 JOIN 的结构大概可以描述成下面这几种，而且还可以嵌套。

- 表 JOIN 表
- （表 JOIN 表） JOIN 表
- 表 JOIN 子查询
- 子查询 JOIN 子查询

还需要关注一下（Sub Query）子查询的部分：

- [13.2.13 Subqueries](https://dev.mysql.com/doc/refman/8.0/en/subqueries.html)

简单的理解，JOIN 和子查询都是构建了一个临时的表，和常规的表不同的是，这里的表名和表里的列都不是固定的，而是动态生成的。

到这里 SELECT 语句的构成分析的就差不多了，下面是对 SELECT 语句的构成进行合理的抽象。

### 分析和抽象

其实抽象的工作是比较容易的，官方文档中的描述已经把抽象给出来了。在这个简单的 ORM 框架里面，大概是下面这几个。

- select_expr 对应列和聚合函数
- table_references 对应表、JOIN、子查询
- where_condition 对应查询条件
- col_name 对应列

这里还需要再看一下关于表达式和聚合函数的部分：

- [9.5 Expressions](https://dev.mysql.com/doc/refman/8.0/en/expressions.html)
- [12.20 Aggregate Functions](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions-and-modifiers.html)

到这里，在这个简单的 ORM 框架里需要实现的部分，就都差不多都了解了，下面就可以开始动手了。

### 构造语句

#### 从最简单的开始

先从最简单的 `SELECT 列 FROM 表` 开始。首先需要构造一个处理 SELECT 语句的对象，封装相关的数据和方法，一般叫它查询构造器（这里叫 SELECT 查询构造器，为了和 INSERT、UPDATE、DELETE 区分开）。

这里上来就会遇到问题，语句里的列和表怎么来。在 ORM 框架中，这里的列和表，不是手动写入的，而是通过解析结构体得到的。既然是通过解析结构体，那么就需要有解析的规则。解析规则由 ORM 框架定义，是解析结构体的属性、还是解析结构体的属性的注释、还是解析 Golang 结构体的属性上的 tag、还是混合模式。

结构体需要按照 ORM 框架定义的解析规则去写，要不然 ORM 框架对着没有按规则来的结构体也是抓瞎的。虽然注释和 tag 可以自定义的随便写，但是结构体它至少会有属性，所以这里可以做一个兜底的策略。这个简单的 ORM 框架里的解析规则为：默认解析结构体的属性，如果结构体的属性上有 tag 就解析 tag。

解析出来的玩意，专业一点的叫法，叫 metadata（元数据），内容是结构体和数据库表的映射关系。包括结构体名和数据库表名的对应关系，结构体的属性名和数据库表的字段名的对应关系，里面还会有一些辅助的信息，这个后面再说。

#### 元数据

在 Golang 里面，可以通过反射解析结构体。通过反射的一些列操作，可以得到结构体名、结构体的属性、还有挂在这些玩意上面的其他信息。这里通过转换结构体名得到数据库表名，拿到结构体名之后，直接驼峰命名转蛇形命名就行。

还可以通过解析注释或者定义接口的方式获取数据库表名。注释就不多说了，通过 AST 解析结构体的注释就行。定义接口，就是定义一个获取数据库表名的接口，然后具体对应数据库表的结构体去实现这个接口。

这样在 SQL 构建的过程中，通过类型断言可以确认结构体是否实现了这个接口，如果实现了这个接口就调用接口取获取自定义的数据库表名。

需要注意的是，元数据里关于结构体的属性名和数据库表的字段名的对应关系的数据，在两个方向上的都需要。因为元数据不止构造 SQL 语句的时候需要正向用，处理结果集的时候也需要用，只不过方向是反过来的，通过数据库字段名，找到对应的结构体属性名。

元数据这块的内容是独立的，可以和 SQL 语句构造的内容隔离开来，自己成为一个独立的模块，这里叫它元数据注册中心。

#### 回到 SQL 语句的构造

解决完元数据的问题，那么语句里的列和表就都有了，这个语句就很好构造了。把传进来的结构体解析之后，把列和表捞出来塞到语句里就行。

对于列来说，后面还可以结合元数据，做一下列存不存在之类的校验。但是如果列写了别名或者是一个复杂查询啥的，校验起来会有点麻烦。

#### 下面搞 WHERE 语句后面的

`SELECT 列 FROM 表 WHERE 查询条件`。

在动手设计之前，需要先观察一下查询条件的结构。这里列举几个常用的查询条件。

- 列{=|>|<|!=}值
- 列=值 AND 列=值
- 列=值 OR 列=值
- 列=值 AND (列=值 OR 列=值)
- 列 LIKE 值
- 列 IN (值)

这里可以观察到，查询条件的结构，大体上是嵌套起来的左中右的结构。中间是操作符，左边是列或者值，右边一般都是值。

这里看上去是一样的，但是实际上有两层抽象。第一层是：左=列，中间=操作符，右边=值；第二层是：左=第一层抽象 中间=操作符，右=第一层抽象。所以这里需要处理的对象不仅有第一层抽象的 column（列）和 value（值）。还有对第一层抽象的抽象，也就是 expression（表达式）。而且可以得出一个结论，列和值都是表达式的一种。由此可以推论，列和值可以是对象，但是表达式肯定是个接口。

这种左中右的结构，可以联想到树的结构，中间是一个根结点，左右是两个叶子结点。大概的形状见图：**orm.drawio.html 2-0**。常规的等于、大于、小于、不等于等数学运算和与、或两个逻辑运算可以直接通过这个结构表示。包括 LIKE 和 IN 也可以用这个结构。问题在于 NOT 逻辑运算和原生语句。NOT 只有一边，原生语句直接没有结构。其实这两个玩意，树形结构也是可以兼容的。

NOT 直接让他没有左边就可以了，处理的时候判断一下左边是不是空的就行。大概的形状见图：**orm.drawio.html 2-2**。

原生语句看上去是自定义的，不符合树的结构，但是可以换个思路，把语句拆开看，其实它已经包含了列和值，所以它就是一个完整的第一层抽象。只不过它只有左边，不需要中间和右边（语句放右边，不需要左边和中间也行）。大概的形状见图：**orm.drawio.html 2-4**。

代码中处理的时候用递归就行，先递归处理左边的非空叶子，然后处理中间的操作符，然后递归处理右边的非空叶子。每次递归的结果，两边加括号包起来，这样不容易出错。遇到空的位置，空格就不管了。括号和空格多了就多了，只要保证语法没问题就行。

基本的 WHERE 用这个结构就差不多了，需要注意 WHERE IN 那里不仅可以直接填数据，还可能涉及到子查询，这后面再说。

#### 下面处理 GROUP BY

`SELECT 列 FROM 表 GROUP BY 列 HAVING 查询条件`。

GROUP BY 其实很简单了，这玩意有两个部分。前面的列很简单了，没啥好说的。后面的 HAVING 和上面的 WHERE 是一样的。官方文档里面这两个位置都是 where_condition 把上面 WHERE 的逻辑直接拿过来复用就可以。

#### 下面处理 ORDER BY

`SELECT 列 FROM 表 ORDER BY 列`。

这玩意也没啥好说的，就是个列，加上升序或者降序的标志。

#### 下面处理 LIMIT 和 OFFSET

`SELECT 列 FROM 表 LIMIT 20 OFFSET 100`。

这两个也没啥好说的，就是两个数字。

#### 处理 SELECT 后面的

- `SELECT 列 FROM 表`。
- `SELECT 列 AS 新名字 FROM 表`。
- `SELECT 聚合函数 FROM 表`。
- `SELECT 聚合函数 AS 新名字 FROM 表`。

SELECT 后面就两大类东西：列和聚合函数，还会涉及到别名。另外，这里还可以插入原生表达式。

从形态上看，列和聚合函数完全不一样，但是它们都可以放在 SELECT 后面，所以这里肯定有一个抽象。其实官方文档里也已经告诉你了，这个抽象叫 select_expr（这里叫查询表达式）。

这里分开处理这两个玩意。列很简单了，就是列。聚合函数需要一个单独的对象，对象里面放上聚合函数的名字还有聚合函数操作的那个列。别名就目前的场景而言其实很简单，就直接对象里给一个字符串设置一下就好了。加上表、JOIN、子查询的别名之后，这里的别名会变的复杂一点。原生表达式就更简单了，直接把表达式原封不动的放在这里就行了。

到这里，在单个表上的操作基本就都搞定了。下面开始搞 JOIN 和子查询。

#### JOIN

`SELECT 列 FROM JOIN`。

实现简单的 JOIN 其实不怎么复杂，主要功能包括起别名、选择列、复杂一点的是 JOIN 可以嵌套。

常用的 JOIN 结构大概就是下面这几个：

- 表 A JOIN 表 B ON ...
- 表 A AS 新名字 JOIN 表 B AS 新名字 ON ...
- 表 A JOIN (表 B JOIN 表 C ON ...) ON ...

之前处理 FROM 后面那个位置的时候是直接用的数据库表名，但是那个位置其实可以放的玩意有表名、JOIN、子查询，这明摆了是要有一个抽象的，官方文档也告诉你了，叫 table_references（这里叫表表达式）。但是这三种对象的处理方式肯定是不一样的，语句形态差的都很远，基本没有共同点，所以铁定是分开各写各的。

观察一下上面的几个 JOIN 的样例，又是个左中右的结构，只不过这里的左右两边是一样的，都是 table_references。处理思路和上面的 WHERE 那里处理查询条件的思路是一样的，都是用递归，先处理递归左边的非空叶子，再递归处理右边的非空叶子。另外 JOIN 还有 ON 子句，官方文档里 ON 后面跟的是 search_condition。它和 WHERE 后面的 where_condition 是差不多的，这里不做复杂处理，所以直接拿过来复用。

这里可以换个思路理解，把带有 JOIN 的查询分成两块。JOIN 单独拿出来看，它最终其实就会变成个临时表。把别名当做表名，左右两个 table_references 的列就是临时表的列，这样问题就又回到了之前处理单个数据库表的问题。后面解决子查询的时候，思路也是这样的。

选择列和上面 SELECT 那里处理查询表达式的地方差不多。但是加入 JOIN 之后，列的处理会变得复杂一点，以前的列都是默认在同一个表里的，加入 JOIN 之后，表就不是一个了。所以列里面，必须把自己属于哪个表存着，这样构造列的时候，需要先构造前面的前缀，而且前缀这玩意可以有别名。也就是表和 JOIN 包括后面的子查询都要有别名，列从原来依托于元数据直接构造变成依托于 table_references 构造。

#### 子查询

子查询看着很复杂，但是构成子查询的元素，上面都已经处理过了。前面说过，子查询就是一个临时表，把别名当做表名，左右两个 table_references 的列就是临时表的列，如果子查询设置了查询表达式，那么临时表的列就是查询表达式里面写的那些列。

整体思路和 JOIN 的思路是差不多的，区别的地方在于对列的处理方式不同。处理 JOIN 的时候，查询表达式里的列可能来自不同的表。但是在子查询这里，所有的列都来自子查询的临时表。

#### 语句构建结束

在编码的时候使用 Builder 设计模式实现，基本思路就是将和 SELECT 语句有关的数据的填充和使用这些数据构造 SELECT 语句的部分隔离开。其最终形态就是常见的 ORM 框架的一堆用起来很流畅的链式调用，最后一个 First() 方法或者 Get() 方法，然后查询构造器构建 SQL、执行 SQL、处理结果集一气呵成之后返回结果集。

到这里简单的 SELECT 语句的构建思路就差不多了。下面就是执行语句和处理结果集了。

### 执行语句

执行语句分为两种，一种是直接执行，一种是事务执行。这两个的区别也很简单就是拿到数据库实例之后，要不要开事务。这个地方可以做两个抽象，一个是数据库实例的抽象，一个是事务的抽象。其中事务的抽象可以包括数据库的抽象，和装饰器有点像，事务对象可以理解成把数据库对象拿过来，用事务功能装饰一下。

### 处理结果集

结果集的处理也不难，前面构造的元数据里不是已经有反方向的映射关系了嘛。先通过数据库返回的结果集里面的结果的字段名，结合元数据，把所有的变量类型先确定好。然按照变量类型构造一个接收数据库返回的结构，接收的时候注意顺序。

成功接收到数据库的返回结果后，先通过元数据构造一个对应的空的结构体。然后通过数据库返回的字段名，结合元数据，反向去找结构体的属性。然后通过反射或者内存操作，把值塞到结构体对应的属性上去。

### 全流程结束

到这里，构造元数据、生成 SELECT 语句、执行语句、处理返回值，就都处理完了。设计上的东西，看类图、流程图会更加直观。细节上的实现，直接看代码，这个靠说是说不太清楚的。