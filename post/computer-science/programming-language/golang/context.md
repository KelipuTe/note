---
draft: false
date: 2023-01-15 08:00:00 +0800
lastmod: 2023-01-15 08:00:00 +0800
title: "Golang 的 context 包的使用"
summary: "context.Background()，context.WithValue()，context.WithCancel()，context.WithDeadline()，context.WithTimeout()"
toc: true

categories:
- golang

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- golang
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> go version go1.19 windows/amd64 

### 资料

- [{demo-golang}](https://github.com/KelipuTe/demo-golang)/demo/context/

context 也叫上下文，可用于携带数据或者控制调度流程。在常见的设计中，会把 context.Context 作为方法的第一个参数。

### 五个常用的方法

#### Background()

这个方法用于创建流程的第一个上下文。后续的上下文，如果没有特殊的情况，都是由它派生出来的。

#### WithValue()

这个方法用于挂载一个键值对到上下文上去。通常用于传递整条链路都需要用的参数。在使用的时候需要注意，WithValue() 的第三个参数的类型是 any。后面通过 Context.Value() 取出来的数据，需要进行类型转换后才能使用。

代码见：{demo-golang}/demo/context/context_test.go 的 f8ContextWithValue()

#### WithCancel()

#### context.WithDeadline()，context.WithTimeout()

这三个方法一起看，它们的用途是差不多的，都是通过控制取消的信号，来控制流程。区别在于 WithCancel() 只能手动控制取消。WithDeadline() 和 WithTimeout() 可以设置到期时间，到期它会自动取消，也可以在到期之前手动控制取消。

代码见：{demo-golang}/demo/context/context_test.go 的 f8ContextWithCancel()

WithCancel() 在上下文取消之前 Context.Done() 和 Context.Err() 是拿不到信号的。取消之后 Context.Done() 可以读到信号，Context.Err() 会返回 context.Canceled。

这里看一下 WithDeadline()。WithTimeout() 里面其实是调了 WithDeadline()。

```
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}
```

代码见：{demo-golang}/demo/context/context_test.go 的 f8ContextWithDeadline() 和 f8ContextWithDeadlineV2()。

使用 WithDeadline() 的时候有一个时序的问题，超时和取消，哪个先发生。这个对 Context.Done() 没有影响，但是对 Context.Err() 是有的。如果超时发生在取消之前，那异常就是 context.DeadlineExceeded。如果超时发生在取消之后，那异常就是 context.Canceled。
