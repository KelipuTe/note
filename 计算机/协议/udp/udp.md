---
draft: false
date: 2022-05-02 08:00:00 +0800
lastmod: 2022-05-02 08:00:00 +0800
title: "UDP"
summary: "UDP"
toc: true

categories:
- protocol(协议)

tags:
- computer-science(计算机科学)
- protocol(协议)
- udp
---

### UDP

UDP（User Datagram Protocol，用户数据报协议）由 IETF（The Internet Engineering Task Force、国际互联网工程任务组） 的 RFC 768 定义。

- [https://www.rfc-editor.org/rfc/rfc768](https://www.rfc-editor.org/rfc/rfc768)
- [https://datatracker.ietf.org/doc/html/rfc768](https://datatracker.ietf.org/doc/html/rfc768)

UDP 是一种无连接的传输层协议。提供面向事务的简单不可靠信息传送服务。

- UDP 协议不保证数据包的可靠性。基于 UDP 协议的网络通信的可靠性由应用层实现。
- UDP 协议的数据包没有顺序，不重传，不进行流量控制，无法得知是否完整到达。

UDP 适用于不需要连接、可随时发送数据或者数据量巨大的场景：如：DNS、TFTP、视频、音频等多媒体通信、广播通信等。

### UDP 报文格式

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

- Source Port：源端口号，16 bit
- Destination Port：目标端口号，16 bit
- Length：包长度，16 bit。保存了 UDP 首部的长度跟数据的长度之和。
- Checksum：校验和，16 bit。是为了提供可靠的 UDP 首部和数据而设计。