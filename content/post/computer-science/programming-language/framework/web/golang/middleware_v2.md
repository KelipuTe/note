---
draft: false
date: 2023-02-11 08:00:00 +0800
lastmod: 2023-02-11 08:00:00 +0800
title: "Golang 实现简单的 Web 框架 -- middleware(中间件)"
summary: "为什么要有中间件；"
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

### 前言

在看这篇之前，建议先看一下 [Golang 实现简单的 Web 框架 -- router(路由)](/post/computer-science/programming-language/framework/web/golang/router_v2)

### 资料

- [{demo-golang}](https://github.com/KelipuTe/demo-golang)/demo/web/middleware/
- <a href="/drawio/computer-science/programming-language/framework/web/web.drawio.html">web.drawio.html</a>

### 前置（后置）工作成对出现

承接上一篇的问题，在前一篇里讨论前置（后置）工作的时候，都是以"前置（后置）工作是可以独立执行的整体"为前提讨论的。如果它们之间有合作关系，也就是需要相互传递数据，那么怎么办呢？这里可以使用封装的思路，设计一个全链路通用的数据结构。然后所有的前置（后置）工作和最终的处理方法都接收这个数据结构作为参数。

在最外层 ServeHTTP 方法最开始的地方创建一个这样的数据结构，先封装 ServeHTTP 方法的两个参数，然后在后面调用到的所有方法那里一层一层传下去。如果后面的工作需要前面的工作的数据，就可以借助这个数据结构进行传递。

```
type 全链路通用的数据结构 struct {
	// ServeHTTP 方法的两个参数
	http.ResponseWriter
	*http.Request
	// 其他参数
}
```

这里沿用上一篇中，访问 `/user/id` 的时候的结果。结合上面说的全链路通用的数据结构，最终地执行逻辑可以总结成下面这样。注意一定是引用传递，值传递进去的是一个副本，方法里面修改副本是不会影响外面的这个本体的。

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	// 构造一个全链路通用的数据结构，封装 ServeHTTP 方法的两个参数
	// 前置工作 user (引用传递全链路通用的数据结构)
	// 前置工作 user-id (引用传递全链路通用的数据结构)
	// `/user/id` 的处理逻辑 (引用传递全链路通用的数据结构)
	// 后置工作 user (引用传递全链路通用的数据结构)
}
```

这个结构可以解决需要互相传递数据的问题。比如，如果"后置工作 user"需要使用"前置工作 user"生成的数据，那就可以借助全链路通用的数据结构，"前置工作 user"在执行的时候往里面塞点东西，这样后面的流程就可以用了。看上去没有什么大的漏洞。

### 处理异常

前面所有的流程里面，都是以代码正常执行为前提进行设计的，尚且没有考虑异常处理，因为异常处理本身确实就比较烦。

有 try-catch 结构的语言，直接一个大的 try-catch 包起来。

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	// 构造一个全链路通用的数据结构，封装 ServeHTTP 方法的两个参数
	try {
	    // 前置工作 user (引用传递全链路通用的数据结构)
	    // 前置工作 user-id (引用传递全链路通用的数据结构)
	    // `/user/id` 的处理逻辑 (引用传递全链路通用的数据结构)
	    // 后置工作 user (引用传递全链路通用的数据结构)
	} catch () {
	}
}
```

没有 try-catch 结构的语言，只能在最后接收参数的地方判断一下返回值的内容，根据返回值的内容进行相应的处理。Golang 在最前面还可以加个 recover 恢复块。

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	// 构造一个全链路通用的数据结构，封装 ServeHTTP 方法的两个参数
	defer func(){ recover() }
	// 前置工作 user (引用传递全链路通用的数据结构)
	// 前置工作 user-id (引用传递全链路通用的数据结构)
	// `/user/id` 的处理逻辑 (引用传递全链路通用的数据结构)
	// 后置工作 user (引用传递全链路通用的数据结构)
	if 全链路通用的数据结构里的标记位 == x {
	    // 标记位等于 x 就怎么怎么样
	} else if 全链路通用的数据结构里的标记位 == y {
	    // 标记位等于 y 就怎么怎么样
	}
}
```

### 屏蔽复杂度

前面的设计已经解决了绝大部分的问题。但是还有一点缺陷，全链路通用的数据结构里面会杂糅大量的数据。这些数据对于后面的整个流程来说是必须的，但是对于其中某一些流程来说，不是必须的。

比如，"前置工作 user"和"后置工作 user"想要进行沟通，所以在全链路通用的数据结构里面放了一个值。但是，这个值对于"前置工作 user-id"和 `/user/id` 的处理逻辑来说就是没用的。

这里还可以优化。怎么优化呢？"前置工作 user"和"后置工作 user"不是想要进行沟通嘛，那么能不能直接把这两个部分连起来，变成类似下面这样的结构呢。这样的话，前面需要放到全链路通用的数据结构里的值是不是就不要了。

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	// 构造一个全链路通用的数据结构，封装 ServeHTTP 方法的两个参数
	func 前置工作 user + 后置工作 user (引用传递全链路通用的数据结构) {
	    // 前置工作 user 的处理逻辑
	    func 剩下的 (引用传递全链路通用的数据结构) {
	        // 前置工作 user-id (引用传递全链路通用的数据结构)
	        // `/user/id` 的处理逻辑 (引用传递全链路通用的数据结构)
	    }
	    // 后置工作 user 的处理逻辑
	}
}
```

怎么实现呢？首先需要注意到，这里需要传进去的已经不是那个全链路通用的数据结构了。需要传进去的是上面的伪代码中，中间的那个部分，也就是下面这部分，它是个方法。

```
func 剩下的 (引用传递全链路通用的数据结构) {
    // 前置工作 user-id (引用传递全链路通用的数据结构)
    // `/user/id` 的处理逻辑 (引用传递全链路通用的数据结构)
}
```

乍一看不怎么好搞。但是，如果补一个"后置工作 user-id"给"前置工作 user-id"，是不是就变得和上面"前置工作 user"和"后置工作 user"一样了。

```
func 剩下的 (引用传递全链路通用的数据结构) {
    // 前置工作 user-id (引用传递全链路通用的数据结构)
    // `/user/id` 的处理逻辑 (引用传递全链路通用的数据结构)
    // （补一个）后置工作 user-id (引用传递全链路通用的数据结构)
}
```

所以，这里可以假设有一个什么都不干的"后置工作 user-id"和它对应，那么整个代码就可以变成这样了，一个类似套娃的结构。

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	// 构造一个全链路通用的数据结构，封装 ServeHTTP 方法的两个参数
	func 前置工作 user + 后置工作 user (引用传递全链路通用的数据结构) {
	    // 前置工作 user 的处理逻辑
	    func 前置工作 user-id + 空的后置工作 user-id (引用传递全链路通用的数据结构) {
	        // 前置工作 user-id
	        // `/user/id` 的处理逻辑 (引用传递全链路通用的数据结构)
	        // （补一个）后置工作 user-id
	    }
	    // 后置工作 user 的处理逻辑
	}
}
```

这样，除了最里面的处理逻辑，外面的前置（后置）工作的部分，就都可以抽象成同一个结构了，是可以随意组合或者调整顺序的。

### 中间件

上面这玩意就是所谓的中间件的概念了，见图：**web.drawio.html 6-2-2**。也就是常听到的洋葱模型，见图：**web.drawio.html 6-2-4**。或者同心圆模型，见图：**web.drawio.html 6-2-5**。

本人更喜欢同心圆模型。前面两个模型的示意图，都没有明显的把嵌套的关系展现出来，更多的展示的是层层递进的关系。而同心圆模型，精准地反映了嵌套的关系，一层一层的进去之后，不管怎么走，都要再一层一层的原路出来。

### 下面用 Golang 实现一下这个结构

首先上面有两个部分需要定义：前置（后置）工作、处理逻辑

```
// HTTPHandleFunc 处理逻辑
type HTTPHandleFunc func(p7ctx *HTTPContext)

// HTTPMiddleware 前置（后置）工作
type HTTPMiddleware func(next HTTPHandleFunc) HTTPHandleFunc
```

具体的处理逻辑，定义成下面这样。

```
func UserId (*HTTPContext) {
    // get user by id
}
```

具体的前置（后置）工作的实现方式，就像下面这样。

```
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

这里假设定义两个前置（后置）工作。

```
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

```
// 最内层的处理逻辑
chain := func UserId()
// 套第一层
mb := BMiddleware()
chain = mb(chain)
// 套第二层
ma := AMiddleware()
chain = ma(chain)
// 执行
chain(ctx)
```

怎么装起来的见图：**web.drawio.html 6-4**。最后的效果等价于下面这样的伪代码。

```
// before AMiddleware
    // before BMiddleware
        serve(p7ctx)
    // after BMiddleware
// after AMiddleware
```

### 可路由的中间件

可路由的中间件就是在路由树的基础上，分别给每个路由树结点设置中间件。这样在匹配过程中，如果结点上设置了中间件，那么就把这些中间件记录下来。在路由匹配到某个路由结点获取到路由的处理方法之后，把最终的处理方法和这些中间件按照定义好的顺序套起来即可。