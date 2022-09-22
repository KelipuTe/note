---
draft: false
title: "使用 Go 实现 Web 框架 -- middleware(中间件)"
date: 2022-09-16 08:00:00 +0800
lastmod: 2022-09-16 08:00:00 +0800
categories:
  - golang
tags:
  - golang
  - web
  - http
  - middleware(中间件)
summary: "中间件的原理和实现方式。"
---

> go version go1.19

### 资料

- {demo-golang}/demo/web/middleware/

### 中间件

web 框架的中间件可以理解成一种 aop 方案的实现。

可以借助洋葱模型、责任链设计模式、链式调用的概念来理解中间件的整体结构。

中间件的结构示例见：**web.drawio.html 4-2**。

定义中间件的时候需要关注两个重要的组成部分：路由的对应的处理方法和中间件的处理方法。

这两个部分定义一起规定了中间件该怎么定义，中间件定义需要围绕这两个定义去实现，要不然调用链条串不起来。

```go
type HTTPHandleFunc func(p7ctx *HTTPContext)
```

```go
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

假设定义两个中间件

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
// 最内层业务代码
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

```
// before AMiddleware
    // before BMiddleware
        serve(p7ctx)
    // after BMiddleware
// after AMiddleware
```
