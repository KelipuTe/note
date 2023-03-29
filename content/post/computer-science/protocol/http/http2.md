---
draft: false
date: 2022-05-07 08:00:00 +0800
lastmod: 2022-05-07 08:00:00 +0800
title: "HTTP/2"
summary: "HTTP/2"
toc: true

categories:
- protocol(协议)

tags:
- computer-science(计算机科学)
- protocol(协议)
- http
---

### HTTP/2

HTTP/2 由 IETF（The Internet Engineering Task Force、国际互联网工程任务组） 的 RFC 7540 定义。

- [https://www.rfc-editor.org/rfc/rfc7540](https://www.rfc-editor.org/rfc/rfc7540)

#### 头信息压缩

HTTP 协议不带有状态，每次请求都必须附上所有信息。所以，请求的很多字段都是重复的。
比如 Cookie 和 User Agent，一模一样的内容，每次请求都必须附带，这会浪费很多带宽，也影响速度。

为此 SPDY 使用的是通用的 DEFLATE 算法，而 HTTP/2 则使用了专门为首部压缩而设计的 HPACK 算法，引入了头信息压缩机制（header compression）。
一方面，头信息使用 gzip 或 compress 压缩后再发送。
另一方面，客户端和服务端同时维护一张头信息表，所有字段都会存入这个表，生成一个索引号。以后就不发送同样字段了，只发送索引号，这样就提高速度了。

#### 二进制协议

HTTP/1.1 版的请求头部肯定是文本（ASCII 编码），请求体可以是文本，也可以是二进制。

HTTP/2 则是一个彻底的二进制协议，头信息和数据体都是二进制。并且统称为 frame（帧）：header frame（头信息帧）和 data frame（数据帧）。

二进制协议的一个好处是，可以定义额外的帧。HTTP/2 定义了近十种帧，为将来的高级应用打好了基础。如果使用文本实现这种功能，解析数据将会变得非常麻烦，二进制解析则方便得多。

```
    +-----------------------------------------------+
    |                 Length (24)                   |
    +---------------+---------------+---------------+
    |   Type (8)    |   Flags (8)   |
    +-+-------------+---------------+-------------------------------+
    |R|                 Stream Identifier (31)                      |
    +=+=============================================================+
    |                   Frame Payload (0...)                      ...
    +---------------------------------------------------------------+
```

- Length：数据帧的长度
- Type：帧类型
- Flags：标志位
- R：流标识符的最高位被保留
- Stream Identifier：流标识符，用来标识 frame（帧）属于哪个 stream（流）
- Frame Payload：数据帧（通过 HPACK 算法压缩过的 HTTP 头部和请求体）

HTTP/2 定义了 10 种类型的帧，一般分为数据帧和控制帧两类：数据帧（0x0~0x2）、控制帧（0x3~0x9）。

| 帧类型 | 类型编码 | 用途 |
| --- | --- | --- |
| DATA | 0x0 | 传递 HTTP 包体 |
| HEADERS | 0x1 | 传递 HTTP 头部 |
| PRIORITY | 0x2 | 指定 stream 流的优先级 |
| RES_STREAM | 0x3 | 终止 stream 流 |
| SETTINGS | 0x4 | 修改连接或者 stream 流的配置 |
| PUSH_PROMISE | 0x5 | 服务端推送资源时描述请求的帧 |
| PING | 0x6 | 心跳检测，兼具计算 RTT 往返时间的功能 |
| GOAWAY | 0x7 | 终止连接或者通知错误 |
| WINDOW_UPDATE | 0x8 | 流量控制 |
| CONTINUATION | 0x9 | 传递较大 HTTP 头部时的持续帧 |

标志位用于携带控制信息：

- END_HEADRES，头数据结束的标志。
- END_STREAM，单方向数据发送结束，后续不会再有数据帧。
- PRIORITY，表示流的优先级。

#### stream（数据流）

HTTP/2 通信都在一个 TCP 连接上完成，这个连接可以承载任意数量的双向数据流。

HTTP/2 将每个请求或响应的所有数据包，称为一个 stream（数据流）。每个数据流都有一个独一无二的编号。另外还规定，发送端发出的数据流，ID 一律为奇数，接收端发出的，ID 一律为偶数。

> 在 Nginx 中，可以通过 http2_max_concurrent_streams 配置来设置 stream 的上限，默认是 128 个。

同一个连接中的 Stream ID 是不能复用的，只能顺序递增，所以当 Stream ID 耗尽时，需要发⼀个控制帧 GOAWAY ，用来关闭 TCP 连接。

- stream 里可以包含 1 个或多个 message，message 对应 HTTP/1 中的请求或响应，由 HTTP 头部和包体构成。
- message 里包含⼀条或者多个 frame，frame 是 HTTP/2 最小单位，以二进制压缩格式存放 HTTP/1 中的内容（头部和请求体）。

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/protocol/http/http2_stream.drawio.png">)

[//]: # (</div>)

HTTP/2 的数据包是不按顺序发送的，同一个连接里面连续的数据包，可能属于不同的请求。因此，必须要对数据包做标记，指出它属于哪个请求。数据包发送的时候，都必须标记数据流 ID，用来区分它属于哪个数据流。

帧的头部会携带 Stream ID 信息，接收端可以通过 Stream ID 有序组装成 HTTP 消息。不同 stream 的 frame 可以乱序发送，但是同一 stream 的 frame 必须严格有序。

HTTP/2 可以取消某一次请求，同时保证 TCP 连接还打开着，可以被其他请求使用。数据流发送到一半的时候，发送端和接收端都可以发送信号（RST_STREAM 帧），取消这个数据流。而 HTTP/1.1 取消数据流的唯一方法，就是关闭 TCP 连接。

发送端还可以指定数据流的优先级。优先级越高，接收端就会越早响应。

#### TCP 连接的多路复用

HTTP/2 的多路复用允许同时通过单一的 TCP 连接发起多重的 "请求--响应" 消息。发送端和接收端都可以同时发送多个请求或回应，而且不用按照顺序一一对应，这样就避免了队头堵塞。

举例来说，在一个 TCP 连接里面，接收端同时收到了 A 请求和 B 请求，于是先回应 A 请求。结果发现处理过程非常耗时，于是就发送 A 请求已经处理好的部分，接着回应 B 请求，完成后，再发送 A 请求剩下的部分。

在过去， HTTP 性能优化的关键并不在于高带宽，而是低延迟。TCP 连接会随着时间进行自我调谐，起初会限制连接的最大速度，如果数据成功传输，会随着时间的推移提高传输的速度。由于这种原因，让原本就具有突发性和短时性的 HTTP 连接变的十分低效。

HTTP/2 通过让所有数据流共用同一个连接，可以更有效地使用 TCP 连接，让高带宽也能真正的服务于 HTTP 的性能提升。这种单连接多资源的方式，减少服务端的连接压力，内存占用更少，连接吞吐量更大。而且由于 TCP 连接的减少而使网络拥塞状况得以改善，同时慢启动时间的减少，使拥塞和丢包恢复速度更快。

##### 问题

HTTP/2 主要的问题在于也在于多个 HTTP 请求复用一个 TCP 连接。下层的 TCP 一旦发生丢包现象，就会触发 TCP 重传机制。这样所有的 HTTP 请求都必须等待这个求了的包被重传回来。

#### 服务端推送

HTTP/2 允许服务端未经请求，主动向客户端发送资源，这叫做 server push（服务端推送）。

常见场景是客户端请求一个网页，这个网页里面包含很多静态资源。正常情况下，客户端必须收到网页后，解析 HTML 源码，发现有静态资源，再发出静态资源请求。

其实，服务端可以预期到客户端请求网页后，很可能会再请求静态资源，所以就主动把这些静态资源随着网页一起发给客户端了。而且服务端推送还有一个很大的优势是可以缓存。这也让在遵循同源的情况下，不同页面之间可以共享缓存资源成为可能。

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/protocol/http/http2_server_push.drawio.png">)

[//]: # (</div>)

服务端在推送资源时，会通过 PUSH_PROMISE 帧传输 HTTP 头部，并通过帧中的 Promised Stream ID 字段告知客户端，接下来会在哪个偶数号 stream 中发送请求体。

如下图，stream 1 表示请求 HTML。服务端可在 stream 1 中通知客户端 CSS 资源即将到来，并告知传输 CSS 资源的 Stream ID 是 2。然后在 stream 2 中发送 CSS 资源。stream 1 和 stream 2 是可以并发的。

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/protocol/http/http2_server_push_stream.drawio.png">)

[//]: # (</div>)