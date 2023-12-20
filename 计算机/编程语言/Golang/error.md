---
draft: false
date: 2022-05-16 08:00:00 +0800
lastmod: 2022-05-16 08:00:00 +0800
title: "error"
summary: "error"
toc: true

categories:
- golang

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- golang
---

> plan for failure, not success

### panic 和 recover

不要用 panic 和 recover 模拟 try 和 catch。

异常和错误是不同的，异常需要处理，意外错误或者致命错误（fatal error）才需要用 panic。

### 定义错误

- 尽可能避免 sentinel error（使用特定的值来表示错误）。
- 尽可能避免 error type（使用结构体来表示错误），虽然它可以提供更多的上下文。

### handle errors once

> You should only handle errors once

错误只应该处理一次，要么记日志，要么报错，只处理一次。

### github.com/pkg/errors

用 github.com/pkg/errors 包装 error。

- errors.New()：创建一个带堆栈信息的错误
- errors.Wrap()：包底层的或者第三方的错误
- errors.Cause()：获取错误最底层的原始错误

具有最高重用性的包，只能返回原始错误或者自定义错误，不应该包装错误。

### Go 1.13

- errors.Is()：可以一层层的 Unwrap() 展开 error 寻找原始错误。
- errors.As()：判断一个错误能不能被转换成指定的错误。
- fmt.Errorf("%w",err)：Errorf 底层会包装 err。

