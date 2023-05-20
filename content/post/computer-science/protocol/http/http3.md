---
draft: true
date: 2022-05-07 08:00:00 +0800
lastmod: 2022-05-07 08:00:00 +0800
title: "HTTP/3"
summary: "HTTP/3"
toc: true

categories:
- protocol(协议)

tags:
- computer-science(计算机科学)
- protocol(协议)
- http
---

HTTP/1.1 中的管道（ pipeline）传输中如果有⼀个请求阻塞了，那么队列后请求也统统被阻塞住了。
HTTP/2 多个请求复用⼀个TCP连接，⼀旦发生丢包，就会阻塞住所有的 HTTP 请求。
这都是基于 TCP 传输层的问题，所以 HTTP/3 把 HTTP 下层的 TCP 协议改成了 UDP 协议。
但是，目前 HTTP/3 普及的很慢。因为 QUIC 是新协议，很多网络设备只会当做 UDP 处理，这会出问题。



基于 UDP 设计的 QUIC 协议可以实现类似 TCP 的可靠性传输。当某个流发生丢包时，只会阻塞这个流，其他流不会受到影响。
TLS 3 升级成了最新的 1.3 版本，头部压缩算法也升级成了 QPack 。
基于 TCP 的 HTTPS 要建立⼀个连接，要花费 6 次交互。先是建立 TCP 3 次握⼿，然后是 TLS/1.3 的 3 次握⼿。
QUIC 直接把以往的 TCP 和 TLS/1.3 的 6 次交互合并成了 3 次。