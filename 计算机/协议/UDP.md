---
draft: false
date: 2022-05-02 08:00:00 +0800
title: "UDP 协议"
summary: "UDP 协议"
toc: true

categories:
  - 协议

tags:
  - 计算机
  - 协议
  - udp
---

## 反向链接

[协议](/计算机/协议/IP)；
[网络间进程间通信](/计算机/网络/网络间进程间通信)；

## 资料

## 正文

### UDP

UDP 是在传输层工作的协议，建立在 IP 之上。

UDP（User Datagram Protocol，用户数据报协议）
是由 IETF（The Internet Engineering Task Force、国际互联网工程任务组） 的 RFC 768 定义的。

[RFC 768](https://www.rfc-editor.org/rfc/rfc768)；
[RFC 768](https://datatracker.ietf.org/doc/html/rfc768)；

UDP 是一种无连接的协议。提供简单的不可靠的信息传输服务。
因为不需要连接，所以 UDP 除了支持一对一的通信，还支持一对多和多对多的通信。

UDP 不保证数据包的可靠性。数据包不重传、没有顺序、不进行流量控制、无法得知是否完整到达。
如果想实现基于 UDP 协议的可靠的通信，那么就需要在应用层设计一些策略来保证通信的可靠性。

UDP 适用于不需要连接、可随时发送数据或者数据量巨大的场景。
比如：DNS、TFTP、视频、音频等多媒体通信、广播通信等。

### 报文格式

```
   0      7 8     15 16    23 24    31
  +--------+--------+--------+--------+
  |     Source      |   Destination   |
  |      Port       |      Port       |
  +--------+--------+--------+--------+
  |                 |                 |
  |     Length      |    Checksum     |
  +--------+--------+--------+--------+
  |
  |          data octets ...
  +---------------- ...
```

UDP 的报文格式非常简单。

- Source Port：源端口号，16 bit；
- Destination Port：目标端口号，16 bit；
- Length：包长度，16 bit；保存了 UDP 首部的长度跟数据的长度之和。
- Checksum：校验和，16 bit；是为了提供可靠的 UDP 首部和数据而设计。

UDP 报文的首部，固定只有 8 字节。

UDP 没有分块机制，UDP 报文的数据段过大时，由 IP 来进行分片处理。
