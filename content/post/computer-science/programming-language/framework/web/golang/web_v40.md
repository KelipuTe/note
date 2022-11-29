---
draft: false
date: 2022-09-24 08:00:00 +0800
lastmod: 2022-09-25 08:00:00 +0800
title: "使用 Golang 实现复杂的 Web 框架"
summary: "使用 net/http 包，实现一个复杂的 Web 框架。"
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
- middleware(中间件)
- golang
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> go version go1.19 windows/amd64

### 前言

在看这篇之前，建议先看下面这几篇：

- 使用 Golang 开启 HTTP 服务
- 使用 Golang 实现简单的 Web 框架 -- router(路由)
- 使用 Golang 实现简单的 Web 框架 -- middleware(中间件)

路由树和中间件是 web 框架的核心。其他的功能，都是在这两个核心的基础上，再增加亿点点细节而已。

### 资料

- {web-framework-go}/v40

### 实现功能

- 主要实现：
  - 路由树（静态、通配符、路径参数、正则表达式）、路由组
  - 全局中间件、可路由的中间件
- 次要实现：
  - 内存池（请求上下文对象复用）
  - 服务管理（管理多个子服务）
  - 优雅退出、退出时的回调方法
- 计划实现：
  - 用户认证（中间件实现）
  - 文件操作（上传、下载）
  - 单元测试、集成测试、性能测试
  - 设计文档

### 路由树

首先是路由树结点的设计。结点的基础数据包括：结点类型、这个路由结点代表的那段路径、命中路由之后的的处理逻辑。

静态路由子结点使用 map 结构存储，查询时直接就可以通过路由段查询子结点。

通配符子结点、路径参数子结点、正则表达式子结点，这三个结点属于特殊结点，而且存在冲突关系，所以单独存储。

为了支持可路由的中间件，路由结点上还需要有存储中间件的地方，这样就可以为每个结点单独设置中间件。

另外，服务在运行的时候只要命中的是同一个路由，那么用到的中间件一定也是相同的，在服务启动的时候就可以把中间件遍历好，然后缓存下来。

```go
// routingNode 路由结点
type routingNode struct {
	// nodeType 结点类型
	nodeType int
	// part 这个路由结点代表的那段路径
	part string
	// path 从根路由到这个路由结点的全路径
	path string

	// f4handler 命中路由之后的处理逻辑
	f4handler HTTPHandleFunc

	// m3routingTree 路由子树，子结点的 path => 子树根结点
	m3routingTree map[string]*routingNode
	// p7paramChild 路径参数结点
	p7paramChild *routingNode
	// paramName 路径参数路由和正则表达式路由，都会提取路由参数的名字
	paramName string
	// p7regexpChild 正则表达式结点
	p7regexpChild *routingNode
	// p7regexp 正则表达式
	p7regexp *regexp.Regexp
	// p7anyChild 通配符结点
	p7anyChild *routingNode

	// s5f4middleware 结点上注册的中间件
	s5f4middleware []HTTPMiddleware
	// s5f4middlewareCache 服务启动后，命中结点时，需要用到的所有中间件
	s5f4middlewareCache []HTTPMiddleware
}
```

路由树的构造和遍历并不复杂，使用递归逻辑处理即可。不用担心递归带来的性能问题。

对于路由树的递归操作，都发生在服务启动时，这个时候会遍历路由树然后将结果缓存下来。

服务启动后，当请求访问过来时，就可以直接使用缓存里的结果，而不用每次都去遍历路由树。

### 路由组

路由组就是个语法糖。相当于路由组方法会在路由组内的每个成员注册的时候，附加路由组的路由前缀和路由组定义的中间件。

```go
// Group 添加一组路由
func (p7this *HTTPHandler) Group(path string, s5f4mw []HTTPMiddleware, s5routeData []RouteData) {
	for _, rd := range s5routeData {
		t4path := path
		if "/" != rd.Path {
			t4path = path + rd.Path
		}
		p7this.addRoute(rd.Method, t4path, rd.F4handle, s5f4mw...)
	}
}
```

### 中间件

全局中间件和可路由中间件是分开放的。全局中间件存储在核心处理逻辑上。可路由中间件存储在路由树结点上。

```go
// HTTPHandlerInterface 核心处理逻辑的接口定义
type HTTPHandlerInterface interface {
	http.Handler
	...
}

// HTTPHandler 核心处理逻辑
type HTTPHandler struct {
	...
	// s5f4middleware 全局中间件
	s5f4middleware []HTTPMiddleware
	...
}
```

当请求访问过来时，第一站到的是核心处理逻辑，核心处理逻辑会完成全局中间件的组装和执行。

```go
func (p7this *HTTPHandler) ServeHTTP(i9w http.ResponseWriter, p7r *http.Request) {
	...
	// 倒过来组装，先组装的在里层，里层的后执行
	// 最里层应该是找路由然后执行业务代码
	t4chain := p7this.doServeHTTP
	for i := len(p7this.s5f4middleware) - 1; i >= 0; i-- {
		t4chain = p7this.s5f4middleware[i](t4chain)
	}
	// 写入响应数据这个中间件应该由框架开发者处理
	// 它是最后一个环节，应该在最外层
	t4m := FlashRespMiddleware()
	t4chain = t4m(t4chain)
	t4chain(p7ctx)
}
```

在通过全局中间件之后，进入查询路由树的步骤。查询结果里会有路由树结点上的可路由中间件。

使用和全局中间件一样的套路，完成一遍可路由中间件的组装和执行。最后调用路由上的处理逻辑，开始真正的业务逻辑。

```go
func (p7this *HTTPHandler) doServeHTTP(p7ctx *HTTPContext) {
	...
	p7ri := p7this.findRoute(p7ctx.P7request.Method, p7ctx.P7request.URL.Path)
	...
	// 这里用同样的套路，处理路由上的中间件，最后执行业务代码
	t4chain := p7ri.p7node.f4handler
	for i := len(p7ri.p7node.s5f4middlewareCache) - 1; i >= 0; i-- {
		t4chain = p7ri.p7node.s5f4middlewareCache[i](t4chain)
	}
	t4chain(p7ctx)
}
```

### 优雅退出

想实现优雅退出，程序就不能阻塞在不可控的位置。这里可以直接把服务丢到协程里去。

然后在最外面，实现一个信号等待的逻辑，这样就可以通过信号控制程序的运行状态。

```go
func (p7this *ServiceManager) Start() {
	// 启动服务
	log.Println("服务启动中。。。")
	for _, p7s := range p7this.s5p7HTTPService {
		t4p7s := p7s
		go func() {
			if err := t4p7s.Start(); nil != err {
				if http.ErrServerClosed == err {
					log.Printf("子服务 %s 已关闭\n", t4p7s.name)
				} else {
					log.Printf("子服务 %s 异常退出，err:%s\r\n", t4p7s.name, err)
				}
			}
		}()
	}
	log.Println("服务启动完成。")
	
	// 监听 ctrl+c 信号
	c4signal := make(chan os.Signal, 2)
	signal.Notify(c4signal, os.Interrupt)
	select {
	case <-c4signal:
		...
	}
}
```

在可以主动介入程序运行之后，就可以设计主动拒绝新请求的逻辑了。这样可以实现服务不完全停止的情况下，拒绝对外服务。

因为服务停止不仅仅是不对外服务这么简单，在服务真正的停止之前，还有很多善后的工作需要做。

```go
// HTTPHandler 核心处理逻辑
type HTTPHandler struct {
	...
	// isRunning 服务是否正在运行
	isRunning bool
}

func (p7this *HTTPHandler) doServeHTTP(p7ctx *HTTPContext) {
	// 如果服务已经关闭了就直接返回
	if !p7this.isRunning {
		p7ctx.I9writer.WriteHeader(http.StatusInternalServerError)
		_, _ = p7ctx.I9writer.Write([]byte("服务已关闭"))
		return
	}
	...
}
```

虽然服务停止前有很多善后的工作需要做，但是理论上不会持续很久。

为了防止意外卡死的情况出现，可以再加一层超时强制停止的逻辑。必要的时候，也可以设计主动强制关闭的入口。

```go
func (p7this *ServiceManager) Start() {
	...
	// 监听 ctrl+c 信号
	c4signal := make(chan os.Signal, 2)
	signal.Notify(c4signal, os.Interrupt)
	select {
	case <-c4signal:
		log.Printf("接收到关闭信号，开始关闭服务，限制 %d 秒内完成。。。\r\n", p7this.shutdownTimeOut/time.Second)
		// 再次监听 ctrl+c 信号
		go func() {
			select {
			case <-c4signal:
				log.Println("再次接收到关闭信号，服务直接退出。")
				os.Exit(1)
			}
		}()
		time.AfterFunc(p7this.shutdownTimeOut, func() {
			log.Println("优雅关闭超时，服务直接退出。")
			os.Exit(1)
		})
		p7this.Shutdown()
	}
}
```

在拒绝新请求之后，由于有可能还有旧的请求没有处理完，所以是不能立刻就关闭服务的，需要等待一段时间。

```go
func (p7this *ServiceManager) Shutdown() {
	...
	log.Println("停止接收新请求。")
	for _, p7hs := range p7this.s5p7HTTPService {
		p7hs.Stop()
	}

	log.Printf("等待正在执行的请求结束，等待 %d 秒。。。", p7this.shutdownWaitTime/time.Second)
	time.Sleep(p7this.shutdownWaitTime)
	...
}
```

服务正式关闭服务之后，可能还有一些收尾的工作需要处理，然后才能彻底退出程序。

比如：系统里如果有缓存的话，可能需要把缓存进行持久化处理；系统关闭时，需要上报数据其他服务等。

这个可以通过回调实现，和中间件的用法类似。不过最后执行的时候，是所有的回调是并发执行的，而不是像中间件一样套起来，一次执行的。

```go
func (p7this *ServiceManager) Shutdown() {
	...
	log.Println("开始执行子服务的关闭回调。。。")
	for _, p7hs := range p7this.s5p7HTTPService {
		log.Printf("执行子服务 %s 的关闭回调，限制 %d 秒内完成。。。", p7hs.name, p7this.shutdownCallbackTimeOut/time.Second)
		for _, f4cb := range p7hs.s5f4shutdownCallback {
			t4f4cb := f4cb
			wg.Add(1)
			go func() {
				defer wg.Done()
				t4ctx, t4cancel := context.WithTimeout(context.Background(), p7this.shutdownCallbackTimeOut)
				defer t4cancel()
				t4f4cb(t4ctx)
			}()
		}
	}
	wg.Wait()
	...
}
```

到这里核心的部分就差不多了，细节上的实现可以看代码。
