---
draft: false
date: 2022-09-17 08:00:00 +0800
lastmod: 2022-09-17 08:00:00 +0800
title: "使用 Golang 实现简单的 Web 框架 -- middleware(中间件)"
summary: "Web 框架中的中间件的原理和实现方式。"
toc: true

categories:
- framework(框架)

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- framework(框架)
- web
- http
- middleware(中间件)
- golang
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> go version go1.19 windows/amd64

### 资料

- [{demo-golang}](https://github.com/KelipuTe/demo-golang)/demo/web/middleware/
- <a href="/drawio/computer-science/programming-language/framework/web/web.drawio.html">web.drawio.html</a>

### 中间件

web 框架的中间件可以理解成一种 AOP 方案的实现。可以借助各路教程中常见的洋葱模型来理解中间件的整体结构，这里我用的是同心圆模型。

中间件的同心圆模型结构示例见图：**web.drawio.html 4-2**。这玩意的核心思路就一个，一层一层的进去，一层一层的出来。另外，需要保证每层的数据格式都是一样的。定义数据格式可以保证，第一顺序可以换，第二可以加层或者减层。

定义中间件的时候需要关注两个重要的组成部分：路由对应的处理方法和中间件的处理方法。

路由对应的处理方法就是，中间件一层一层的进去之后，最里面那层和业务衔接的地方的定义。这里需要处理的是，把中间件的数据格式解包，交给业务逻辑去处理。拿到处理的结果后，在包装成中间件的数据格式，然后丢出去。

中间件的处理方法就是，中间件最外面一层的外面，进入中间件的地方的定义。因为进了第一层之后，每层都是一样的，所以这里只关注最外面一层。这里需要处理的是，把请求数据包装成中间件的数据格式，然后丢进去。拿到处理的结果后，把结果处理成请求想要的响应数据，然后响应回去。

这两个部分定义一起规定了中间件该怎么定义，中间件定义需要围绕这两个定义去实现，要不然调用链条串不起来。

```go
// HTTPHandleFunc 路由对应的处理方法的定义
type HTTPHandleFunc func(p7ctx *HTTPContext)

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

这里假设定义两个中间件。

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
