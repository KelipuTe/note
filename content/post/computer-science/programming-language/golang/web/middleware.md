---
draft: false
date: 2022-09-17 08:00:00 +0800
lastmod: 2022-09-17 08:00:00 +0800
title: "使用 Golang 实现 Web 框架 -- middleware(中间件)"
summary: "中间件的原理和实现方式。"

categories:
- golang

tags:
- computer-science(计算机科学)
- golang
- web
- http
- middleware(中间件)
---

> go version go1.19 windows/amd64

### 资料

- [{demo-golang}](https://github.com/KelipuTe/demo-golang)/demo/web/middleware/
- <a href="/drawio/computer-science/programming-language/golang/web.drawio.html">web.drawio.html</a>

### 中间件

web 框架的中间件可以理解成一种 aop 方案的实现。

可以借助洋葱模型、责任链设计模式、链式调用的概念来理解中间件的整体结构。

中间件的结构示例见图：**web.drawio.html 4-2**。

定义中间件的时候需要关注两个重要的组成部分：路由的对应的处理方法和中间件的处理方法。

这两个部分定义一起规定了中间件该怎么定义，中间件定义需要围绕这两个定义去实现，要不然调用链条串不起来。

```go
// HTTPHandleFunc 路由对应的处理方法的定义
type HTTPHandleFunc func(p7ctx *HTTPContext)
```

```go
// HTTPMiddleware 中间件的处理方法的定义
type HTTPMiddleware func(next HTTPHandleFunc) HTTPHandleFunc
```

具体的中间件的实现方式，就像下面这样。

```go
func DemoMiddleware() HTTPMiddleware {
	return func(next HTTPHandleFunc) HTTPHandleFunc {
		return func(p7ctx *HTTPContext) {
			// before DemoMiddleware
			next(p7ctx)
			// after DemoMiddleware
		}
	}
}
```

假设定义两个中间件。

```go
func AMiddleware() HTTPMiddleware {
	return func(next HTTPHandleFunc) HTTPHandleFunc {
		return func(p7ctx *HTTPContext) {
		    // before AMiddleware
			next(p7ctx)
			// after AMiddleware
		}
	}
}

func BMiddleware() HTTPMiddleware {
	return func(next HTTPHandleFunc) HTTPHandleFunc {
		return func(p7ctx *HTTPContext) {
			// before BMiddleware
			next(p7ctx)
			// after BMiddleware
		}
	}
}
```

然后这么链起来，B 在内层，A 在外层。

```go
// serve 是最内层业务代码
chain := serve
// 组装中间件
mb := BMiddleware()
chain = mb(chain)
ma := AMiddleware()
chain = mb(chain)
// 执行
chain(ctx)
```

最后的效果等价于下面这样的伪代码。

```go
// before AMiddleware
    // before BMiddleware
        serve(p7ctx)
    // after BMiddleware
// after AMiddleware
```

### 可路由的中间件

可路由的中间件就是在路由树的基础上，分别给每个路由树结点设置中间件。

这样在路由匹配到某个路由结点之后，不仅可以获取到路由的处理方法，还可以获取到路由上设置的中间件。

然后把这些中间件，按照定义好的顺序，套起来即可。
