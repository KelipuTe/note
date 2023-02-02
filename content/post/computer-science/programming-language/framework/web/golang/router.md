---
draft: false
date: 2022-09-10 08:00:00 +0800
lastmod: 2022-09-10 08:00:00 +0800
title: "Golang 实现简单的 Web 框架 -- router(路由)"
summary: "路由树的原理和实现方式。"
toc: true

categories:
- framework(框架)

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- framework(框架)
- web
- http
- router(路由)
- golang
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> go version go1.19 windows/amd64

### 资料

- [{demo-golang}](https://github.com/KelipuTe/demo-golang)/demo/web/router/
- <a href="/drawio/computer-science/programming-language/framework/web/web.drawio.html">web.drawio.html</a>

### 路由

在实现路由树之前，首先需要理解路由是干什么的。路由的作用简单地说就是，通过某个请求的请求路径找到对应的处理方法然后处理这个请求。

简单的路由可以使用 map 结构，就是单纯的字符串匹配。map 的 key 就是路由的路径，value 就是路由的处理方法。

像这样 `"user/num" => func userNum()`。如果路由命中了 `user/num`，就调用 `func userNum()` 去处理。

这个结构很简单了，接收到请求时，用请求的路径直接去 map 里找有没有对应的处理方法。能找到就处理，找不到就找不到，就 404。

简单的路由匹配可以解决，但是复杂一点的功能。比如：路径参数路由、正则匹配路由、通配符路由这样的需求，map 结构就没办法整。

比如 `"user/:id" => func userId()` 这样的，我 `:id` 的位置可以变，这 key 要怎么写，不好搞了嘛。

所以对于后面那些功能，就不能用 map 这种强一对一的结构。就需要换一个思路，观察一下 `user/:id` 和 `user/num`。

这两个的差别在于后面的部分，前面是一样的 `user`，这可以联想到分叉的结构。比如：一棵树的两个树杈、一根树枝上的两个叶子。

所以这里就可以考虑用树形结构来改造一下，把 key 拆了，公共的部分作为根结点，不同的部分作为叶子结点。

这样变成树结构之后，路由的解析步骤，就变成一层一层的了，就可以满足上面那些功能的要求，当一对一匹配不到的时候，就可以看看有没有别的路可以走。

也就是路由树的结点在存储静态子结点的同时，还可以额外设置特殊子结点。在找不到静态路由时，可以继续判断有没有特殊结点可以选择。

比如上面的 `user/:id` 和 `user/num`，当来一个 `user/1` 的时候，我直接打，命不重任何一个。

但是 `:id` 是一个特殊结点，它可以判断 `1` 是不是满足它的匹配条件，满足匹配条件的就放它过，也就是路由匹配上了。

路由树的结构示例见图：**web.drawio.html 2-2**。

路由树的设计没有强制的要求，不是非的像图里那样。按照实际场景的需求，只要设计出路由树的构建和查询规则合理即可。

### 路由组

路由组其实就是额外提供的方便使用的接口而已，相当于对路由的注册功能做了一层包装。

路由组的内部，会把定义到路由组上的路由路径前缀和中间件，附加到路由组内的每一个路由上去。
