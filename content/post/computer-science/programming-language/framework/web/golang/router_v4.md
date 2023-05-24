---
draft: false
date: 2023-03-12 08:00:00 +0800
lastmod: 2023-03-12 08:00:00 +0800
title: "【实验性质，带图片】Golang 实现简单的 Web 框架 -- router(路由)"
summary: "Web 框架是什么；路由是什么；路由树是怎么来的；"
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
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- go version go1.19 windows/amd64

## 资料

- [{demo-golang}](https://github.com/KelipuTe/demo-golang)/demo/web/router/
- <a href="/drawio/computer-science/programming-language/framework/web/web.drawio.html">web.drawio.html</a>

前置笔记：[Golang 开启 HTTP 服务](/post/computer-science/programming-language/golang/http)

## 正文

### Web 框架是什么

#### 概念解释

简单的理解，Web 框架就是工程师们在反复开发 WEB 系统的后端服务时，对于 "如何处理重复的代码" 这个问题，想出的一种解决方案。Web 框架的核心目标主要有：减少重复的代码，提高代码的可维护性；提供常用的组件，提高编程的效率。

重复的代码分为两种。一种是，在一个工程中，多处都需要使用的代码。另一种是，在多个工程中，都需要使用的代码。Web 框架内部的设计是为了解决在一个工程中的重复的代码，Web 框架本体的设计是为了解决在在多个工程中的重复的代码。

在一个工程中的重复的代码是指，在一个工程中，有多处的逻辑是相同的，但是编码实现的时候，没有对逻辑进行封装，而是暴力的把代码复制粘贴。这么做会导致，如果这块逻辑有变化，那么所有复制粘贴的地方都需要修改。

在多个工程中都需要使用的代码是指，在多个项目中，可能都需要路由、中间件、参数校验、日志等常用的组件。这个时候就可以把这些常用的组件打包到一起，下次在需要构建类似的工程的时候，就可以直接拿过来用。

#### 比喻解释

用比喻来理解，Web 框架就相当于毛坯房外加一堆事先准备好的建材。毛坯房是指，房屋的框架结构有了但是里面没有装修。事先准备好的建材是指，在房子外面事先准备好了砖头、水泥、石灰这类最基础的材料。毛坯房本体直接就可以用，如果使用者有别的需要，也对房屋进行不同的装修，添加不同的内饰。

比如，烘焙店和服装店内部装修肯定是不一样的。在进行装修的时候，可以直接使用事先准备好的建材。如果觉得事先准备好的建材不能满足需求，那就需要自行准备新的建材。比如，烘焙店和服装店如果想加一面墙，那可以直接用事先准备好的砖头、水泥。但是如果烘焙店想铺木地板或者服装店想贴墙纸，那就需要自行准备了。

这套逻辑套到 Web 框架上来是一样的，毛坯房对应的就是框架的核心组件，事先准备好的建材对应的就是常用的组件。对于 Web 系统来说，主要任务都是处理网络请求（毛坯房本体）。但是具体到不同的业务系统，它们内部处理的具体的业务逻辑肯定是不同的（不同的装修、不同的内饰）。比如，商城系统主要是处理商品和订单的，成绩管理系统主要是处理成绩的。但是有可能会使用相同的组件（事先准备好的建材）。比如，都需要登录、都需要记日志。

#### 本人想到的最合理的解释

如果通过前面的概念和比喻还是不怎么好理解什么是 WEB 框架，那么还有一个更简单的理解，**工程师们想偷懒，他们通过创造并使用 Web 框架这个工具，来达到偷懒的目的**。

### Web 框架是怎么被创造出来的

现在是知道有 Web 框架这么个工具了，但是这个工具是怎么被创造出来的呢？想要解释这个问题，就必须回到 Web 框架这个概念还没有出现的 "蛮荒时代" 去。下面将结合代码来说明这个问题，代码将会使用 Golang 处理 HTTP 1.1 的请求。

### 从 TCP 开始

别看见 TCP 就紧张，这里不要求深入了解 TCP 的细节，只需要知道 TCP 是一个字节流协议就可以了。字节流协议，它的意思就是说，传输数据的时候，它是以字节为单位的，传输的过程像流水一样，一个字节接着一个字节的。见图：**web.drawio.html 2-2**。

![图片](/image/computer-science/programming-language/framework/web/web.2-2.drawio.png)

HTTP 1.1 是基于 TCP 协议实现的，所以 HTTP 1.1 也可以说是一种字节流协议。这里为什么要说这个呢？主要是想强调一下，在计算机的 "眼里"，HTTP 报文它不是一个整体，而是一个一个字符。HTTP 1.1 报文大概的格式见图：**web.drawio.html 2-4-2、2-4-4**。

![图片](/image/computer-science/programming-language/framework/web/web.2-4-2.drawio.png)

![图片](/image/computer-science/programming-language/framework/web/web.2-4-4.drawio.png)

既然发出去的 HTTP 报文就是一串字符，那么如果不说明这串字符怎么解读，那这串字符就毫无意义。所以客户端和服务端会通过约定请求行的 method 和 url 来区分不同的报文，method 和 url 的组合就是路由。

可以区分不同的报文之后，后面的请求头部和请求体才有意义。分析 HTTP 报文是什么类型，然后根据类型提取报文中的数据的过程就是解析 HTTP 报文的过程。

### Golang 的 "net" 包和 "net/http" 包

在 net 包里，提供了对 TCP 的支持。而 "net/http" 包就是基于 net 包，实现了对 HTTP 协议的解析。这里就不费劲的从 TCP 开始搞了，怎么使用 TCP 和解析 HTTP 报文对理解 Web 框架其实没啥帮助。所以直接从 "net/http" 包开始。

再继续之前，建议先看一下 **前置笔记：Golang 开启 HTTP 服务**。下面会直接从 "net/http"包 的 Handler 接口的 ServeHTTP 方法切入。在 ServeHTTP 方法的第二个参数 "*Request" 里面，就可以拿到已经解析好的 HTTP 请求。

```
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

### 从最简单的场景开始

假设，所有接口的处理逻辑都是一个完整的整体。

如果啥都不考虑，那大可以直接用 if-else 的结构去判断 method 和 url，然后在 if-else 分支里面写各自的处理逻辑。如果把处理逻辑封装到方法面去，那这样的代码的逻辑其实是非常清晰的。 

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	if Request.Method == "GET" {
		if Request.URL == "/user/id" {
			// /user/id 的处理逻辑
		} else if Request.URL == "/user/name" {
			// /user/name 的处理逻辑
		}
	} else if Request.Method == "POST" {
		if Request.URL == "/order/create" {
			// /order/create 的处理逻辑
		} else if Request.URL == "/order/delete" {
			// /order/delete 的处理逻辑
		}
	}
}
```

即使后面报文的类型越来越多，无非也就是 if-else 的分支多了一点。可以再对 url 的处理做一次封装，代码就可以拆开来了。

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	if Request.Method == "GET" {
		handleGet()
	} else if Request.Method == "POST" {
		handlePost()
	}
}

func handleGet(){
	if Request.URL == "/user/id" {
		// /user/id 的处理逻辑
	} else if Request.URL == "/user/name" {
		// /user/name 的处理逻辑
	} else if xxx {
	} else if xxx {
	}
}

func handlePost(){
	if Request.URL == "/order/create" {
		// /order/create 的处理逻辑
	} else if Request.URL == "/order/delete" {
		// /order/delete 的处理逻辑
	} else if xxx {
	} else if xxx {
	}
}
```

就目前来看这种给 method 和 url 都写一个 if-else 的分支的写法，似乎并没有什么非常明显的弊端。这种结构也可以使用 map 结构代替。可以是一层的：`map[method+url]处理方法`，或者嵌套的：`map[method]{map[url]处理方法}`。

### 处理逻辑需要一些前置（后置）工作

但是实际上并不是 "所有接口的处理逻辑都是一个完整的整体"。比如，商城系统的下订单和取消订单的接口。下订单的接口的处理逻辑假设为，校验用户+下订单。取消订单的接口的处理逻辑假设为，校验用户+取消订单。

这里可以换一个角度，把下订单的接口的处理逻辑分成：前置工作 "校验用户" 和处理逻辑 "下订单"。把取消订单的接口的处理逻辑分成：前置工作 "校验用户" 和处理逻辑 "取消订单"。

这里讨论的是前置（后置）工作相同的情况。如果每个处理逻辑需要执行的前置（后置）工作不同，那就退回到 "所有接口的处理逻辑都是一个完整的整体" 那种情况去了。前置（后置）工作相同的情况有几种：所有的处理逻辑都需要，部分处理逻辑需要。

#### 所有的处理逻辑都需要

这种场景非常好处理，直接在最外面写就好了。哪怕前置（后置）工作有多个或者它们之间有顺序要求的场景。

```
func ServeHTTP(http.ResponseWriter, *http.Request) {
	// 前置工作 a
	// 前置工作 b
	if Request.Method == "GET" {
		handleGet()
	} else if Request.Method == "POST" {
		handlePost()
	}
	// 后置工作 c
}
```

#### 部分处理逻辑需要

这种场景乍一看可以沿用 "所有的处理逻辑都需要" 的方案，把前置（后置）工作下放到每个 if-else 的分支里面去就好了。但是这么做是有问题的。

假设 "/user/id" 和 "/user/name" 都需要 "前置工作 user"，"/user/id" 自己还需要 "前置工作 user-id"，"/user/name" 自己还需要 "后置工作 user-name"。如果按照上面的写法，代码会写成下面这样子。

```
func handleGet(){
	if Request.URL == "/user/id" {
		// 前置工作 user
		// 前置工作 user-id
		// /user/id 的处理逻辑
	} else if Request.URL == "/user/name" {
		// 前置工作 user
		// /user/name 的处理逻辑
		// 后置工作 user-name
	}
}
```

乍一看问题不大，但是如果后面有 "/user/a"、"/user/b" 一直到 "/user/z" 呢？如果要求每个 "/user/" 开头的，都需要 "前置工作 user" 呢？这时候上面这种写法，"前置工作 user" 就要写 n 次，**很麻烦**。如果后面要求变了，要求每个 "/user/" 开头的都需要 "前置工作 user2"，或者这里就需要改 n 个地方，**更麻烦**。

所以要想办法，把这个公共的模块提取出去。比如，变成下面这样（这里写了个伪代码，用的是 SQL 的 LIKE 语法），把有这种要求的 url 前缀单独拿到一个 if-else 的分支里去。

```
func handleGet(){
	if Request.URL like "/user/%" {
		// 前置工作 user
		if Request.URL == "/user/id" {
			// 前置工作 user-id
			// /user/id 的处理逻辑
		} else if Request.URL == "/user/name" {
			// /user/name 的处理逻辑
			// 后置工作 user-name
		}
		// 后置工作 user
	} else {
		if Request.URL == "/order/id" {
			// /order/id 的处理逻辑
		}
	}
}
```

乍一看问题不大，如果要求每个 "/user/" 开头的从都需要 "前置工作 user" 变成都需要 "前置工作 user2"，那么只需要改一处。但是如果前缀的层级很多呢？比如，加一个 "/user/info/a"，同时要求每个 "/user/info/" 开头的从都需要 "前置工作 user-info"。那上面这种写法就要变成。

```
func handleGet(){
	if Request.URL like "/user/%" {
		// 前置工作 user
		if Request.URL like "/user/info/%" {
			// 前置工作 user-info
			if Request.URL == "/user/info/a" {
				// /user/info/a 的处理逻辑
			}
		} else {
			if Request.URL == "/user/id" {
				// 前置工作 user-id
				// /user/id 的处理逻辑
			} else if Request.URL == "/user/name" {
				// /user/name 的处理逻辑
				// 后置工作 user-name
			}
		}
		// 后置工作 user
	} else {
		if Request.URL == "/order/id" {
			// /order/id 的处理逻辑
		}
	}
}
```

这样下去 if-else 的层级就会变得越来越深了。if-else 的层级太深，不利于代码的可读性和可维护性。更重要的是，这样的代码改的时候，找起来非常的麻烦，还要担心层级会不会改错了。所以这个问题是需要规避的。那么怎么规避呢？可以使用有层次的结构。

设想一个场景，图书馆里有大量的书需要分类管理。那么可以先按书内容的类别，分到不同的楼层去。然后再按书的作者，分到不同的书架上去。如果想处理某一类的书，可以只在某个楼层内操作，不会影响到别的楼层。如果想处理某一类的某个作者的书，可以只在某个楼层的某个书架上操作，不会影响到这个楼层的别的书架，更不会影响到别的楼层。

这里把这种结构画出来就很直观了。见图：**web.drawio.html 4-2**。

![图片](/image/computer-science/programming-language/framework/web/web.4-2.drawio.png)

这种结构在数据结构里对应的就是树形结构。把上面的 method、url、前置（后置）工作对应进去再画一张图。见图：**web.drawio.html 4-4**。

![图片](/image/computer-science/programming-language/framework/web/web.4-4.drawio.png)

### 路由树

#### 基本概念

到这里，所谓的路由树的概念就呼之欲出了。上面的那棵树的结点中记录了全路径，但是这其实是不需要的。当命中 'like "/user/%"' 分支往下走的时候，后面的 url 最前面的那段就肯定是 "/user/"。所以这里就可以借助前缀的思路，用 "/" 作为分隔标志，将上面的那棵树转化成前缀树。见图：**web.drawio.html 4-6-2**。注意有个根结点（"/" 结点）。

![图片](/image/computer-science/programming-language/framework/web/web.4-6-2.drawio.png)

这样当一个 HTTP 请求过来的时候，先通过 method 判断应该到哪一棵路由树里去找。然后用 "/" 将 url 分开，依次去路由树里匹配，如果结点上有前置（后置）工作就需要记录下来。最后找到目标结点时，按照前置工作、处理逻辑、后置工作的顺序依次执行。

比如，在 **web.drawio.html 4-6-2** 这颗树里，访问 "/user/id" 的时候。依次会访问："/" 结点；"user" 结点，记录下 "前置工作 user" 和 "后置工作 user"；id 结点，记录下 "前置工作 user-id" 和 "/user/id" 的处理逻辑。见图：**web.drawio.html 4-6-4**。

![图片](/image/computer-science/programming-language/framework/web/web.4-6-4.drawio.png)

执行的时候，从逻辑上考虑的话，应该是前面的结点的前置任务应该在前面，前面的结点的后置任务应该在后面。所以上面就应该按照 "前置工作 user"、"前置工作 user-id"、"/user/id" 的处理逻辑、"后置工作 user" 的顺序依次执行。

#### 路由树的设计（Golang）

##### 代码结构设计

路由树，从名字里就可以知道，它首先是一个是树结构，所以，设计工作应该从树结点开始。这里用 **web.drawio.html 4-6-2** 里的那个路由树为例。

一条完整的路由会被拆成有父子关系的一系列结点。那么，树结点里面一定有一个数据用于标记它是路由中的哪一个部分。然后，还需要知道父子关系，所以，结点需要存储其子结点的信息。这里一般没有回溯的需求，如果需要支持路由的回溯匹配，那么结点就需要记录父结点的信息。

一条路由最后会对应一个处理方法，这个显然是需要记录的。另外，还有前置工作和后置工作，这部分属于中间件，放到后面说，这里先不搞。所以，代码结构目前应该差不多像下面这样。

```
// routingNode 路由结点
type routingNode struct {
	// part 这个路由结点代表的那段路径
	part string

	// handler 命中路由之后的处理逻辑
	handler HTTPHandleFunc

	// routingTree 路由子树，子结点的 path => 子树根结点
	routingTree map[string]*routingNode
}
```

##### 测试用例设计

对于最基本的路由树结构，测试用例设计是非常简单的。

| 用例编号 | 测试用例 | 测试目标 |
| -- | -- |------|
| 2 | /user | 根结点（空结点） -> 根结点有 1 个子结点（user）   |
| 4 | /user + /user/info | user 结点（空结点） -> user 结点有 1 个子结点（info）    |
| 6 | /user + /order | 根结点有 1 个结点（user） -> 根结点有 2 个结点（user、order）    |

这样最基本的几种情况就都测到了。

#### 路径参数路由、正则匹配路由、通配符路由

静态的路由匹配可以用上面的 if-else 分支或者路由树解决。但是一些高级玩法，比如：路径参数路由、正则匹配路由、通配符路由，这样的就没办法整。

路径参数路由，一般长这样 "user/:id"，能把路由里的某一段提取出来，放到事先定义好的变量里面。

正则匹配路由一般长这样 "user/:id(/\\d/)"，它和路径参数路由很像，区别在于，正则匹配路由在提取出路由里的某一段之后，还需要对数据的格式进行校验。

通配符路由一般长这样 "/user/*"，它就很暴力了，只要路由前面是 "/user/" 开头的，后面是什么都可以匹配上。

这里以路径参数路由为例。比如，定义 "user/:id" 这样一个路径参数路由，在对应的处理方法里，事先定义好了一个变量 id 用于接收 ":id" 这个位置对应的字符串。

最终达到的效果是：如果访问的是 "user/1"，那么变量 id 的值就是字符串 "1"；如果访问的是 "user/2"，那么变量 id 的值就是字符串 "2"；如果访问的是 "user/a"，那么变量 id 的值就是字符串 "a"。

对于这种 ":id" 的位置可以变的路由。if-else 分支或者 map 是无解的。因为，":id" 对应的位置可变，意味着这里会对应无穷多个 if-else 分支。路由树有没有解呢？路由树可以解决，路由树只需要在 user 结点上增加一个特殊的 ":id" 结点，专门用于处理路径参数路由即可。见图：**web.drawio.html 4-8-2**。

![图片](/image/computer-science/programming-language/framework/web/web.4-8-2.drawio.png)

访问 "/user/1" 的时候。依次会访问："/" 结点；"user" 结点。到了 "user" 结点之后，按照默认逻辑下面要找的是 1 结点，但是 "user" 结点下面只有 "info" 结点、"id" 结点，无法匹配。这个时候就可以尝试匹配 ":id" 结点，看看 1 符不符合 ":id" 结点的要求。

这里 ":id" 的位置可以是任意的字符串，所以 1 是符合要求的，所以 "/user/1" 最终调用的就应该是 ":id" 结点的处理逻辑。见图：**web.drawio.html 4-8-4**。

![图片](/image/computer-science/programming-language/framework/web/web.4-8-4.drawio.png)

但是需要注意的是，这三个玩意匹配的时候可能都能匹配上，所以需要人为的定义这三个特殊的路由，哪个优先匹配，哪个最后匹配。

最后，写代码的时候，路由注册和路由查询的逻辑建议分开。因为路由注册和路由查询的逻辑看似都是找结点，但是细节上还是有点区别的。路由注册的时候，是严格按照路由层级注册的，而路由查询的时候，需要考虑特殊结点。

#### 路径参数路由、正则匹配路由、通配符路由的设计（Golang）

##### 代码结构设计

路径参数路由、正则匹配路由、通配符路由的匹配逻辑和普通结点是不同的，需要能识别出它们。在树结构里，可以通过给结点打上标记来区分不同的结点。

在上面的设计中，把所有的结点都放到了一个 map 里面管理，这个操作在这里是行不通的。这三种特殊的结点没有实际上的 url，也就提取不出 map 的 key。所以，这三种特殊的结点需要专门的地方来存储。

而且这三种特殊结点理论上都是唯一的。比如，一个结点上不可能同时有两个路径参数路由。假设，同时存在 "/user/:id" 和 "/user/:name"，那么 "/user/a" 应该命中哪一个呢。另外，通配符结点的后面不应该有子结点，因为，全部都会被通配符结点截胡。

普通结点里用于标记它是路由中的哪一个部分的那个参数，在这三种结点里面是没啥用的。路径参数路由和正则匹配路由需要一个额外的参数存储参数的名字，正则匹配路由还需要一个参数存储正则表达式。

所以，上面的代码结构需要增加点东西，目前应该差不多像下面这样。

```
// routingNode 路由结点
type routingNode struct {
	// nodeType 结点类型
	nodeType int
	// part 这个路由结点代表的那段路径
	part string
	// path 从根路由到这个路由结点的全路径
	path string

	// handler 命中路由之后的处理逻辑
	handler HTTPHandleFunc

	// routingTree 路由子树，子结点的 path => 子树根结点
	routingTree map[string]*routingNode
	// paramChild 路径参数结点
	paramChild *routingNode
	// paramName 路径参数路由和正则表达式路由，都会提取路由参数的名字
	paramName string
	// regexpChild 正则表达式结点
	regexpChild *routingNode
	// regexp 正则表达式
	regexp *regexp.Regexp
	// anyChild 通配符结点
	anyChild *routingNode
}
```

##### 测试用例设计

三种特殊结点的测试用例设计就稍微复杂一点了。而且因为结点类型变多了，还需要考虑注册结点的时候的顺序的问题。三种特殊结点有可能会干扰注册普通结点时的结点查询逻辑。

| 用例编号 | 测试用例                                                                 | 测试目标                                                                         |
|------|----------------------------------------------------------------------|------------------------------------------------------------------------------|
| -    | 普通结点 + 参数路由结点                                                        | -                                                                            |
| 2    | /user + /user/:id                                                    | user 结点（空结点） -> user 结点有 1 个参数路由子结点（:id）                                     |
| 4    | /user + /user/info + /user/:id                                       | user 结点有 1 个子结点（info） -> user 结点有 1 个子结点（info）和 1 个参数路由子结点（:id）              |
| 6    | /user + /user/:id + /user/info                                       | user 结点有 1 个参数路由子结点（:id） -> user 结点有 1 个子结点（info）和 1 个参数路由子结点（:id）           |
| 8    | /user + /user/:id + /user/:id/info                                   | 参数路由结点（:id）（空结点） -> 参数路由结点（:id） 有 1 个子结点（info）</br>**注意用例 8 和用例 6 的区别。**     |
| 10   | /user + /user/:id + /user/:name                                      | user 结点有两个参数路由子结点，**报错**                                                     |
| 12   | /user + /user/:id + /user/:id/:name                                  | 参数路由结点（:id）（空结点） -> 参数路由结点（:id）有 1 个参数路由子结点（:name）                           |
| -    | 普通结点 + 正则表达式结点                                                       | -                                                                            |
| 42   | /user + /user/:id(/\\d/)                                             | user 结点（空结点） -> user 结点有 1 个正则表达式子结点（:id）                                    |
| 44   | /user + /user/info + /user/:id(/\\d/)                                | user 结点有 1 个子结点（info） -> user 结点有 1 个子结点（info）和 1 个正则表达式子结点（:id）             |
| 46   | /user + /user/:id(/\\d/) + /user/info                                | user 结点有 1 个正则表达式子结点（:id） -> user 结点有 1 个子结点（info）和 1 个正则表达式子结点（:id）         |
| 48   | /user + /user/:id(/\\d/) + /user/:id(/\\d/)/info                     | 正则表达式结点（:id）（空结点） -> 正则表达式结点（:id） 有 1 个子结点（info）</br>**注意用例 48 和用例 46 的区别。** |
| 50   | /user + /user/:id(/\\d/) + /user/:name(/^\[A-Za-z0-9\]+$/)           | user 结点有两个正则表达式子结点，**报错**                                                    |
| 52   | /user + /user/:id(/\\d/) + /user/:id(/\\d/)/:name(/^\[A-Za-z0-9\]+$/) | 正则表达式结点（:id）（空结点） -> 正则表达式结点（:id）有 1 个正则表达式子结点（:name）                        |
| -    | 普通结点 + 通配符结点                                                         | -                                                                            |
| 82   | /user + /user/*                                                      | user 结点（空结点） -> user 结点有 1 个通配符子结点                                           |
| 84   | /user + /user/info + /user/*                                         | user 结点有 1 个子结点（info） -> user 结点有 1 个子结点（info）和 1 个通配符子结点                    |
| 86   | /user + /user/* + /user/info                                         | user 结点有 1 个通配符子结点 -> user 结点有 1 个子结点（info）和 1 个通配符结点子结点                     |
| 88   | /user + /user/* + /user/*/info                                       | 通配符结点后面不能有子结点，**报错**                                                         |
| -    | 4 种结点混合，这里就不一个一个写了，场景太多了                                             | -                                                                            |

### 前置（后置）工作成对出现

上面讨论前置（后置）工作的时候，都是以 "前置（后置）工作是可以独立执行的整体" 为前提讨论的。如果它们之间有合作关系，也就是需要相互传递数据，那么怎么办呢？这个放到下一篇里说：[【实验性质，带图片】Golang 实现简单的 Web 框架 -- middleware(中间件)](/post/computer-science/programming-language/framework/web/golang/middleware_v4)

## 参考（reference）

- {极客时间}/[Go 实战训练营](https://u.geekbang.org/subject/go2nd)
  - Web 框架部分
