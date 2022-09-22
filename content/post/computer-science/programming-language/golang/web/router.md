---
draft: false
title: "使用 Go 实现 Web 框架 -- 路由"
date: 2022-09-15 08:00:00 +0800
lastmod: 2022-09-15 08:00:00 +0800
categories:
  - golang
tags:
  - golang
  - web
  - http
  - router(路由树)
summary: "路由树的的原理和实现方式。"
---

> go version go1.19

### 资料

- {demo-golang}/demo/web/router/
- <a href="/drawio/computer-science/programming-language/golang/web.drawio.html">web.drawio.html</a>

### 路由

路由的作用是通过请求的路径找到对应的处理方法。

简单的路由可以使用 map 结构，就是单纯的字符串匹配。map 的 key 就是路由的路径，value 就是路由的处理方法。

接收到请求时，用请求的路径直接去 map 里找有没有对应的处理方法。但是这种结构无法满足路径参数路由、正则匹配路由、通配符路由这样的需求。

这些需求需要使用到路由树结构，路由树的结点在存储静态子结点的同时，还可以额外设置特殊子结点。在找不到静态路由时，可以继续判断有没有特殊结点可以选择。

路由树的结构示例见：**web.drawio.html 2-2**。

路由树的设计没有强制的要求，按照实际场景的需求，合理的设计路由树的构建和查询规则即可。