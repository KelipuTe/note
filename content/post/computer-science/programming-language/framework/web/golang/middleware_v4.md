---
draft: false
date: 2023-03-12 08:00:00 +0800
lastmod: 2023-03-12 08:00:00 +0800
title: "【实验性质，带图片】Golang 实现简单的 Web 框架 -- middleware(中间件)"
summary: "中间件是什么；"
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
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- go version go1.19 windows/amd64

## 资料

- [{demo-golang}](https://github.com/KelipuTe/demo-golang)/demo/web/middleware/
- <a href="/drawio/computer-science/programming-language/framework/web/web.drawio.html">web.drawio.html</a>

前置笔记：[【实验性质，带图片】Golang 实现简单的 Web 框架 -- router(路由)](/post/computer-science/programming-language/framework/web/golang/router_v4)

## 正文

### 前置（后置）工作成对出现

承接上一篇的问题，在前一篇里讨论前置（后置）工作的时候，都是以 "前置（后置）工作是可以独立执行的整体" 为前提讨论的。如果它们之间有合作关系，也就是需要相互传递数据，那么怎么办呢？这里可以使用封装的思路，设计一个全链路通用的数据结构。然后所有的前置（后置）工作和最终的处理方法都接收这个数据结构作为参数。

在最外层 ServeHTTP 方法最开始的地方创建一个这样的数据结构，先封装 ServeHTTP 方法的两个参数，然后在后面调用到的所有方法那里一层一层传下去。如果后面的工作需要前面的工作的数据，就可以借助这个数据结构进行传递。

```
type 全链路通用的数据结构 struct {
	// ServeHTTP 方法的两个参数
	http.ResponseWriter
	*http.Request
	// 其他参数
}
```

这里沿用上一篇中，访问 "/user/id" 的时候的结果。结合上面说的全链路通用的数据结构，最终地执行逻辑可以总结成下面这样。注意一定是引用传递，值传递进去的是一个副本，方法里面修改副本是不会影响外面的这个本体的。

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	// 构造一个全链路通用的数据结构，封装 ServeHTTP 方法的两个参数
	// 前置工作 user (引用传递全链路通用的数据结构)
	// 前置工作 user-id (引用传递全链路通用的数据结构)
	// /user/id 的处理逻辑 (引用传递全链路通用的数据结构)
	// 后置工作 user (引用传递全链路通用的数据结构)
}
```

这里画个代码块的示意图，见图：**web.drawio.html 6-6-2**，下面要用。

![图片](/image/computer-science/programming-language/framework/web/web.6-6-2.drawio.png)

这个结构可以解决需要互相传递数据的问题。比如，如果 "后置工作 user" 需要使用 "前置工作 user" 生成的数据，那就可以借助全链路通用的数据结构，"前置工作 user" 在执行的时候往里面塞点东西，这样后面的流程就可以用了。看上去没有什么大的漏洞。

### 处理异常

前面所有的流程里面，都是以代码正常执行为前提进行设计的，尚且没有考虑异常处理，因为异常处理本身确实就比较烦。

有 try-catch 结构的语言，直接一个大的 try-catch 包起来。

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	// 构造一个全链路通用的数据结构，封装 ServeHTTP 方法的两个参数
	try {
	    // 前置工作 user (引用传递全链路通用的数据结构)
	    // 前置工作 user-id (引用传递全链路通用的数据结构)
	    // /user/id 的处理逻辑 (引用传递全链路通用的数据结构)
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
	// /user/id 的处理逻辑 (引用传递全链路通用的数据结构)
	// 后置工作 user (引用传递全链路通用的数据结构)
	if x == 全链路通用的数据结构里的标记位 {
	    // 标记位等于 x 就怎么怎么样
	} else if y == 全链路通用的数据结构里的标记位 {
	    // 标记位等于 y 就怎么怎么样
	}
}
```

### 屏蔽复杂度

前面的设计已经解决了绝大部分的问题。但是还有一点缺陷，全链路通用的数据结构里面会杂糅大量的数据。这些数据对于后面的整个流程来说是必须的，但是对于其中某一些流程来说，不是必须的。

比如，"前置工作 user" 和 "后置工作 user" 想要进行沟通，所以在全链路通用的数据结构里面放了一个值。但是，这个值对于 "前置工作 user-id" 和 "/user/id" 的处理逻辑来说就是没用的。

那么这里怎么优化呢？"前置工作 user" 和 "后置工作 user" 不是想要进行沟通嘛，那么能不能直接把这两个部分直接连起来呢？可以的，把中间的 "前置工作 user-id" 和 '"/user/id" 的处理逻辑' 看作一个整体的代码块。

那么这里需要的，就是以一整个方法作为参数的结构。代码（伪代码）就会变成类似下面这样的结构。这样的话，前面需要放到全链路通用的数据结构里的值是不是就不要了。

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	// 构造一个全链路通用的数据结构，封装 ServeHTTP 方法的两个参数
	func 前置工作 user + 后置工作 user (引用传递全链路通用的数据结构) {
	    // 前置工作 user 的处理逻辑
	    func 剩下的 (引用传递全链路通用的数据结构) {
	        // 前置工作 user-id (引用传递全链路通用的数据结构)
	        // /user/id 的处理逻辑 (引用传递全链路通用的数据结构)
	    }
	    // 后置工作 user 的处理逻辑
	}
}
```

上面那个代码块示意图会变成这样，见图：**web.drawio.html 6-6-4**。

![图片](/image/computer-science/programming-language/framework/web/web.6-6-4.drawio.png)

这个结构怎么实现呢？首先需要注意到，这里需要传进去的已经不是那个全链路通用的数据结构了。需要传进去的是上面的伪代码中，中间的那个 "func 剩下的" 部分，也就是下面这部分。它是一个整体的代码块，或者说是一个完整的方法。

```
func 剩下的 (引用传递全链路通用的数据结构) {
    // 前置工作 user-id (引用传递全链路通用的数据结构)
    // /user/id 的处理逻辑 (引用传递全链路通用的数据结构)
}
```

这个部分还有一个问题，"func 剩下的" 里面的 "前置工作 user-id" 不是处理逻辑的一部分。换句话说，上面这个操作只是个 "前置工作 user" 和 "后置工作 user" 恰好成对出现的特例，无法推广出一个通用的操作技巧。那么怎么办呢？

这里的 "前置工作 user-id" 如果能和 "前置工作 user" 一样，有一个 "后置工作 user-id" 和它配对是不是就行了。所以，补一个 "后置工作 user-id" 给 "前置工作 user-id"，这个 "后置工作 user-id" 什么都不做，单纯就是占个位置。

同样的如果只有后置工作，也可以补一个什么都不做的前置工作。这样就变得和上面 "前置工作 user" 和 "后置工作 user" 一样了。

```
func 剩下的 (引用传递全链路通用的数据结构) {
    // 前置工作 user-id (引用传递全链路通用的数据结构)
    // /user/id 的处理逻辑 (引用传递全链路通用的数据结构)
    // （补一个）后置工作 user-id (引用传递全链路通用的数据结构)
}
```

然后，整个代码就可以变成这样了，一个类似套娃的结构。上面那个代码块示意图会变成这样，见图：**web.drawio.html 6-6-6**。

![图片](/image/computer-science/programming-language/framework/web/web.6-6-6.drawio.png)

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	// 构造一个全链路通用的数据结构，封装 ServeHTTP 方法的两个参数
	func 前置工作 user + 后置工作 user (引用传递全链路通用的数据结构) {
	    // 前置工作 user 的处理逻辑
	    func 前置工作 user-id + 空的后置工作 user-id (引用传递全链路通用的数据结构) {
	        // 前置工作 user-id
	        // /user/id 的处理逻辑 (引用传递全链路通用的数据结构)
	        // （补一个）后置工作 user-id
	    }
	    // 后置工作 user 的处理逻辑
	}
}
```

这样，除了最里面的处理逻辑，外面的前置（后置）工作的部分，就都可以抽象成同一个结构了，是可以随意组合或者调整顺序的。

### 中间件

上面这玩意就是所谓的中间件（Middleware）的概念了，见图：**web.drawio.html 6-2-2**。也就是常听到的洋葱模型，见图：**web.drawio.html 6-2-4**。或者同心圆模型，见图：**web.drawio.html 6-2-6**。

![图片](/image/computer-science/programming-language/framework/web/web.6-2-2.drawio.png)

![图片](/image/computer-science/programming-language/framework/web/web.6-2-4.drawio.png)

![图片](/image/computer-science/programming-language/framework/web/web.6-2-6.drawio.png)

本人更喜欢同心圆模型。前面两个模型的示意图，都没有明显的把嵌套的关系展现出来，更多的展示的是层层递进的关系。而同心圆模型，精准地反映了嵌套的关系，一层一层的进去之后，不管怎么走，都要再一层一层的原路出来。

### 中间件的设计（Golang）

#### 代码结构设计

上面有两个部分需要定义：前置（后置）工作和处理逻辑。全链路通用的数据结构，上面已经定义过了，这里直接用。

```
// HTTPHandleFunc 处理逻辑
type HTTPHandleFunc func(ctx *HTTPContext)

// HTTPMiddleware 前置（后置）工作
type HTTPMiddleware func(next HTTPHandleFunc) HTTPHandleFunc
```

处理逻辑没什么好说的，接收全链路通用的数据结构就行了。重点在于前置（后置）工作，它需要一个处理逻辑作为输入，然后再返回一个新的处理逻辑。

新的处理逻辑内部需要调用传入的那个处理逻辑，形成一个套娃的结构。具体的前置（后置）工作的实现方式，就像下面这样。

```
func DemoMiddleware() HTTPMiddleware {
	return func(next HTTPHandleFunc) HTTPHandleFunc {
		return func(ctx *HTTPContext) {
			// before DemoMiddleware
			next(ctx)
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

假设，具体的处理逻辑，定义成下面这样。

```
func UserId (p7ctx *HTTPContext) {
    // get user by id
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

![图片](/image/computer-science/programming-language/framework/web/web.6-4.drawio.png)

```
// before AMiddleware
    // before BMiddleware
        serve(p7ctx)
    // after BMiddleware
// after AMiddleware
```

#### 测试用例设计

这种结构没办法直接进行测试，可以使用一些间接方法。比如，让每个中间件都输出一个全局唯一的而且没有前缀冲突的字符串到全链路通用的数据结构里面。整个链路运行之后，从全链路通用的数据结构里面取出字符串，和测试用例相比较。下面举个例子。

```
func UserId (p7ctx *HTTPContext) {
    p7ctx.TestString = append(p7ctx.TestString, "UserId;")
}
```

```
func AMiddleware() HTTPMiddleware {
	return func(next HTTPHandleFunc) HTTPHandleFunc {
		return func(p7ctx *HTTPContext) {
		    p7ctx.TestString = append(p7ctx.TestString, "beforeA;")
			next(p7ctx)
			p7ctx.TestString = append(p7ctx.TestString, "afterA;")
		}
	}
}

func BMiddleware() HTTPMiddleware {
	return func(next HTTPHandleFunc) HTTPHandleFunc {
		return func(p7ctx *HTTPContext) {
			p7ctx.TestString = append(p7ctx.TestString, "beforeB;")
			next(p7ctx)
			p7ctx.TestString = append(p7ctx.TestString, "afterB;")
		}
	}
}
```

##### 测试用例 2

```
chain := func UserId()
mb := BMiddleware()
chain = mb(chain)
ma := AMiddleware()
chain = ma(chain)
chain(ctx)
```

结果："beforeA;beforeB;UserId;afterB;afterA;"

##### 测试用例 4

```
chain := func UserId()
ma := AMiddleware()
chain = ma(chain)
mb := BMiddleware()
chain = mb(chain)
chain(ctx)
```

结果："beforeB;beforeA;UserId;afterA;afterB;"

### 可路由的中间件

上面实现的那个就是可路由的中间件。它就是在路由树的基础上，分别给每个路由树结点设置中间件。这样在匹配过程中，如果结点上设置了中间件，那么就把这些中间件记录下来。在路由匹配到某个路由结点获取到路由的处理方法之后，把最终的处理方法和这些中间件按照定义好的顺序套起来即可。

## 参考（reference）

- {极客时间}/[Go 实战训练营](https://u.geekbang.org/subject/go2nd)
  - Web 框架部分
