---
draft: false
title: "使用 Golang 开启 HTTP 服务"
date: 2022-09-03 08:00:00 +0800
lastmod: 2022-09-03 08:00:00 +0800
categories:
  - golang
tags:
  - golang
  - web
  - http
summary: "几种使用 net/http 包或者 net 包开启 HTTP 服务的方法。"
---

> go version go1.19

### 资料

- {demo-golang}/demo/web/http/

### 开启 HTTP 服务

在 Go 中有多种方式，可以开启 HTTP 服务。但是总的来说基本就下面两大类思路（本质上其实是一类）。

- 使用官方提供的封装好的 `net/http` 包。
- 直接使用 `net` 包从 TCP 开始自行实现。

使用 `net/http` 包的时候，需要关注的最核心的部分，就是 Handler 接口 (src/net/http/server.go)。

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

开启 HTTP 服务和 HTTP 请求被处理的流程大致如下：

- 直接或间接地创建 Server 结构体 (src/net/http/server.go) 的实例。
- 调用 Server 的 ListenAndServe() 方法。
- ListenAndServe() 调用 net.Listen() (src/net/dial.go)，启动 TCP 服务。
- net.Listen() 返回一个 net.Listener 接口 (src/net/dial.go) 的实例。
- 调用 net.Listener 的 Accept() 方法，就可以获取连接上来的 TCP 连接。
- 新开启一个协程，把这个 TCP 连接丢进去处理。自己则继续监听有没有 TCP 连接。
- 处理流程继续往下，会遇到这行代码：`serverHandler{c.server}.ServeHTTP(w, w.req)`。
- 这里调用的就是 Handler 的 ServeHTTP() 方法。再往下就进入框架或者业务处理流程了。
