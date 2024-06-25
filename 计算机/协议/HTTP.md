---
draft: false
title: "HTTP 协议"
summary: "HTTP 协议；"
toc: true

categories:
  - 协议

tags:
  - 计算机
  - 协议

date: 2022-05-06 08:00:00 +0800
---

## 反向链接

[TCP](/计算机/协议/TCP)；

## 正文

- HTTP：Hyper Text Transfer Protocol，超文本传输协议

### HTTP

HTTP 是基于 TCP、IP 协议的应用层网络通讯协议。
它主要规定了发送端和接收端之间的通信格式（报文格式），不涉及数据包（packet）的传输。
它是一个简单的 "请求-响应" 协议，默认使用 80 端口。

HTTP 报文被接收后，在内存中就是一堆字节。
因为 HTTP 是基于 TCP 的，而 TCP 是字节流的。

### 不同版本的 HTTP 协议的架构示意图

| HTTP/1.1 |  HTTPS  |  HTTP/2  |   -    | HTTP/3 |    -     |
|:--------:|:-------:|:--------:|:------:|:------:|:--------:|
|   HTTP   |  HTTP   |   HTTP   |   -    |  HTTP  |    -     |
|    \|    |   \|    |  HPack   | Stream | QPack  |  Stream  |
|    \|    | SSL/TLS | TLS 1.2+ |   -    |  QUIC  | TLS 1.3+ |
|   TCP    |   TCP   |   TCP    |   -    |  TCP   |    -     |
|    IP    |   IP    |    IP    |   -    |   IP   |    -     |
|   MAC    |   MAC   |   MAC    |   -    |  MAC   |    -     |

### HTTP/0.9

该版本极其简单，只有一个 GET 命令。
协议规定，接收端只能响应 HTML 格式的字符串，不能响应别的格式。
接收端响应报文发送完毕，就立刻关闭 TCP 连接。

请求报文：

```
GET /index.html
```

响应报文：

```
<html>
  <body>hello, world</body>
</html>
```

### HTTP/1.0

HTTP/1.0 增加了许多内容。

- 1、改变请求和响应的格式。除了数据部分，每次通信都必须包括 HTTP Header（头信息）。
- 2、除了 GET 命令，还引入了 POST 命令和 HEAD 命令。
- 3、任何格式的内容都可以发送。如，文字、图像、视频、二进制文件等。
- 其他的新增功能还包括：Status Code（状态码）、Content-Type、Content-Encoding 等。

HTTP/1.0 规定，头信息必须是 ASCII 码，后面的数据可以是任何格式。
发送端请求的时候，可以使用 Accept 字段声明自己可以接受哪些数据格式。
接收端响应的时候，必须使用 Content-Type 字段告诉发送端，数据是什么格式。

由于发送的数据可以是任何格式，因此可以把数据压缩后再发送。
发送端在请求时，使用 Accept-Encoding 字段说明自己可以接受哪些压缩方法。
接收端响应的时候，使用 Content-Encoding 字段说明数据的压缩方法。

请求报文：

```
GET /index.html HTTP/1.0
Accept: */*
```

响应报文：

```
HTTP/1.0 200 OK 
Content-Type: text/html
Content-Length: 46

<html>
  <body>hello, world</body>
</html>
```

### HTTP/1.0 的问题

HTTP/1.0 的主要问题是连接无法复用。
每个 TCP 连接只能发送一个请求，接收端的响应报文发送完毕，就关闭连接。
如果要请求其他资源，就必须再建立一个 TCP 连接。

在高延迟的场景下 TCP 的三次握手影响明显。
TCP 的慢启动机制对文件类大数据量的请求影响较大（TCP 分包）。

有些浏览器在请求时，用了一个非标准的字段："Connection: keep-alive"。
这个字段要求服务端不要关闭 TCP 连接，以便其他请求复用。
客户端发起请求时带上这个字段，服务端同样回应这个字段。
这样一个可以复用的 TCP 连接就建立了，直到客户端或服务器主动关闭连接。

### GET 与 POST 的区别

从报文的角度：

HTTP 只是规定传输内容的格式，HTTP 通过 TCP 传输，TCP 依靠 IP 寻址。
GET 和 POST 只是 HTTP 协议中两种通信格式，本质上他们并没有实质的区别。

不带参数时，最大区别就是第一行的方法名 GET 和 POST 不同。
带参数时，在约定中，GET 方法的参数应该放在 url 中，POST 方法参数应该放在 body 中。
但是，这也只是约定，只要服务端能把数据解析出来，改成下面这样也是可以的。

```
POST /index.php?name@xxx&age@18 HTTP/1.1
```

从安全的角度：

HTTP 只是规定传输内容的格式，HTTP 通过 TCP 传输，TCP 依靠 IP 寻址。
在整个过程中，不只是 GET 和 POST，其他的方法也一样，传输内容都是明文的。
也就是，任何的中间人，都可以进行拦截，获取请求和响应的数据。
想要安全传输就必须，在使用 TCP 之前，对要传输的数据进行加密（HTTPS 等）。

### GET 方法的数据长度限制

HTTP 协议没有 url 和 body 的长度限制。
大多数对 url 的限制都是浏览器和服务端做的。
因为太大的数据量对浏览器和服务端都是很大负担。

### POST 产生两个 TCP 数据包

HTTP 协议中没有明确的说明 POST 会产生两个 TCP 数据包。
header 和 body 分开发送，一般是程序自发的行为。

比如，Linux curl 在 POST 请求要传输的数据大于 1024 字节时，
curl 命令行工具并不会直接就发起 POST 请求，而是会分为俩步：

- 1、发送一个只有请求头部的请求，请求头部包含 "Expect:100-continue" 字段，询问服务端使用愿意接受数据。
- 2、当接收到服务端响应的对 "Expect:100-continue" 字段的应答以后，才会把 body 发送给服务器。

但是，并不是所有的服务端都会正确应答 100-continue 字段。
比如，lighttpd（轻量级 HTTP 服务器），就会返回 417 Expectation Failed ，造成错误。

### HTTP/1.1

HTTP/1.1 的定义在 RFC 2616 里面。

[RFC 2616](https://www.rfc-editor.org/rfc/rfc2616);

- HTTP/1.1 新增了许多动词方法：PUT、DELETE、PATCH。
- 新增了身份认证、状态管理、Cache 缓存等机制相关的字段。
  新增的 Range 和 Content-Range 字段用于断点续传。
- 新增 Host 字段，用来指定服务器的域名。
  有了 Host 字段，就可以将请求发往同一台服务器上的不同网站。

#### HTTP/1.1 请求报文结构

<table>
<tr align="center">
<td>请求行（一行）</td>
<td>method</td><td>空格</td><td>url（路由和查询参数）</td><td>空格</td><td>HTTP 版本号</td><td>\r\n</td>
</tr>
<tr align="center">
<td rowspan="2">请求头（多行）</td>
<td colspan="2">键</td><td>: 空格</td><td colspan="2">值</td><td>\r\n</td>
</tr>
<tr align="center">
<td colspan="2">键</td><td>: 空格</td><td colspan="2">值</td><td>\r\n</td>
</tr>
<tr align="center">
<td colspan="7">\r\n（一行）</td>
</tr>
<tr align="center">
<td>请求体</td><td colspan="6">各种格式的数据</td>
</tr>
</table>

- Content-Type，请求体数据类型。

请求体需要通过 Content-Type 字段判断是那种数据格式（json，xml 等）

- Content-Length，请求体长度。

在 HTTP/1.0 中，Content-Length 字段不是必需的。
因为浏览器发现服务器关闭了 TCP 连接，就表明收到的数据包已经全了。
但是在 HTTP/1.1 中，一个 TCP 连接现在可以传送多个报文。
当 TCP 连接连续接收多个 HTTP 报文时。
Content-Length 字段可以确定某个 HTTP 报文请求体的结尾在哪里。
这样就不会误读下一个 HTTP 报文的头部，造成数据混乱。

使用 Content-Length 字段的前提条件是，接收端发送回应之前，必须知道回应的数据长度。
对于一些很耗时的动态操作来说，这意味着，接收端要等到所有操作完成，才能发送数据，显然这样的效率不高。

更好的处理方法是，产生一块数据，就发送一块，采用 stream（流）模式取代 buffer（缓存）模式。
因此，HTTP/1.1 规定可以不使用 Content-Length 字段，而使用 chunked transfer encoding（分块传输编码）。

只要请求或者响应的头部信息有 Transfer-Encoding 字段。
`Transfer-Encoding: chunked`，就表明回应将由数量未定的数据块组成。

每个非空的数据块之前，会有一个 16 进制的数值，表示这个块的长度。
最后是一个大小为 0 的块，就表示本次回应的数据发送完了。

#### HTTP/1.1 请求报文解析

一般通过两个连续的 `\r\n` 判断请求头和请求体分隔的位置。

- 请求头内部每行的数据都通过 `\r\n` 分隔。
- 请求体的长度通过请求头的 Content-Length 字段获取，然后就可以截取请求体的数据了。

#### HTTP/1.1 响应报文结构

<table>
<tr align="center">
<td>状态行（一行）</td>
<td>HTTP 版本号</td><td>空格</td><td>状态码</td><td>空格</td><td>状态码短句</td><td>\r\n</td>
</tr>
<tr align="center">
<td colspan="7">除了状态行，其余部分和请求报文差不多</td>
</tr>
</table>

HTTP 五大类状态码：

- 1xx，提示信息；
- 2xx，成功；
- 3xx，重定向；
- 4xx，发送端错误；
- 5xx，接收端错误；

#### 持久连接

HTTP/1.1 最大的变化，是引入了 persistent connection（持久连接）。
即 TCP 连接默认不关闭，可以被多个请求复用，不用声明 "Connection: keep-alive"。

发送端和接收端发现对方一段时间没有活动，就可以主动关闭连接。
不过，规范的做法是，发送端在最后一个请求时，发送 "Connection: close"，明确要求接收端关闭 TCP 连接。

目前，对于同一个域名，大多数浏览器允许同时建立 6 个持久连接。

#### 管道机制

HTTP/1.1 引入了 pipe lining（管道机制）。
即在同一个 TCP 连接里面，发送端可以同时发送多个请求。
这样就进一步改进了 HTTP 协议的效率。

举例来说，发送端需要请求两个资源。
以前的做法是，在同一个 TCP 连接里面，先发送 A 请求，然后等待接收端做出回应，收到后再发出 B 请求。
管道机制则是允许发送端同时发出 A 请求和 B 请求，但是接收端还是按照顺序，先回应 A 请求，然后回应 B 请求。

### HTTP/1.1 的问题

#### 队头堵塞

虽然 HTTP/1.1 允许复用 TCP 连接，但是同一个 TCP 连接里面，所有的数据通信是按次序进行的。
服务器只有处理完一个回应，才会进行下一个回应。要是前面的回应特别慢，后面就会有许多请求排队等着。
这称为 head-of-line blocking（队头堵塞）。

为了避免这个问题，只有两种方法：一是减少请求数，二是同时多开持久连接。
这导致了很多的网页优化技巧。比如，合并脚本和样式表、将图片嵌入 CSS 代码、domain sharding（域名分片）等。

#### 无状态

- 无状态的好处是，接收端不需要记录状态信息。
- 无状态的坏处是，关联性的操作每次都需要验证（可以借助 Cookie 解决）。

#### 明文传输

- 明文传输的好处是方便解析，省掉了加密的时间。
- 明文传输的坏处是，明文的数据不安全。
