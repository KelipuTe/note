---
draft: false
date: 2022-11-21 08:00:00 +0800
lastmod: 2022-11-21 08:00:00 +0800
title: "使用 Golang 实现简单的 ORM 框架 -- 其他辅助工具"
summary: "ORM 框架中的中间件的原理和实现方式。"
toc: true

categories:
- framework(框架)

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- framework(框架)
- orm
- middleware(中间件)
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

### 中间件

ORM 框架的中间件和 web 框架的中间件的原理是一样的，这里就不重复了。这里需要考虑的是中间件注册在哪里，查询构造器上显然是不合适的，这里可以考虑和方言抽象一样，放到数据库抽象上去。在执行查询之前，先去数据库实例上把中间件拿出来，然后把执行查询的逻辑放在最里面。

另外，需要定义中间件入参和出参的数据格式。这里和 web 框架一样，要不然套不起来。但是这里比 web 框架会复杂一点，在 web 框架那里入参和出参是一样的，整条链路上都只需要处理 TCP 连接的输入流和输出流。ORM 框架这里，输入的参数有 4 种，即 4 种查询构造器。输出的参数有两种，即数据库执行 SQL 返回的两大类结果。

所以需要把 4 中查询构造器封装到中间件入参里面，入参包括查询构造器的类型和查询构造器的本体。把数据库执行 SQL 返回的两大类结果封装到中间件出参里面，出参包括数据库执行 SQL 返回的两大类结果以及异常。这里又有一个问题了，SQL 语句执行完了，怎么处理返回回来的被封装的两大类结果。

可以注意到，SELECT 和 INSERT、UPDATE、DELETE 是分别对应数据库执行 SQL 返回的两类结果的，而且前面执行 SQL 的时候也是分别使用 query() 和 exec() 的。所以在最外面，也就是进入中间件链条的位置，是可以知道执行的到底是哪类 SQL 语句的。所以对于数据库返回的结果，就可以在这里进行类型断言。
