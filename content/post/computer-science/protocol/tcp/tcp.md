---
draft: false
date: 2022-05-02 08:00:00 +0800
lastmod: 2022-05-02 08:00:00 +0800
title: "TCP"
summary: "TCP"
toc: true

categories:
- protocol(协议)

tags:
- computer-science(计算机科学)
- protocol(协议)
- tcp
---

### TCP

TCP（Transmission Control Protocol、传输控制协议）由 IETF（The Internet Engineering Task Force、国际互联网工程任务组） 的 RFC 793 定义。

- [https://www.rfc-editor.org/rfc/rfc793](https://www.rfc-editor.org/rfc/rfc793)
- [https://datatracker.ietf.org/doc/html/rfc793](https://datatracker.ietf.org/doc/html/rfc793)

TCP 是一种面向连接的、可靠的、基于字节流的传输层通信协议，为连接到计算机通信网络的计算机中的成对进程之间提供可靠的通信服务。适用于需要建立连接，保证数据交付的场景，如：HTTP、FTP 等。

字节流形式的数据像水流一样，没有边界，但是有顺序。如果前面的字节没收到，那么后面的字节一定也没收到。

TCP 负责保障数据的可靠性。因为 IP 不能保证数据交付、不能保证数据按序交付、不能保证数据完整性，是不可靠的。

### Header Format（头部格式）

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          Source Port          |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Sequence Number                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Acknowledgment Number                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Data |           |U|A|P|R|S|F|                               |
   | Offset| Reserved  |R|C|S|S|Y|I|            Window             |
   |       |           |G|K|H|T|N|N|                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Checksum            |         Urgent Pointer        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             data                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- Source Port：源端口号，16 bit
- Destination Port：目标端口号，16 bit
- Sequence Number：序列号，32 bit
- Acknowledgment Number：确认应答号，32 bit
- Data Offset：首部长度，4 bit
- Reserved：保留位，6 bit
- URG、ACK、PSH、RST、SYN、FIN：控制位，1*6 bit
- Window：窗口大小，16 bit
- Checksum ：校验和，16 bit
- Urgent Pointer：紧急指针，16 bit
- Options、Padding：选项和填充，长度可变
- data：数据，长度可变

为了网络设备硬件设计和处理方便，首部长度需要是 4 字节的整数倍。

#### 序列号

在建立连接时，会生成一个随机数作为序列号的初始值，通过 SYN 报文传给对端。后续每发送一次数据，就累加一次该数据字节数的数值，以此解决数据包乱序的问题.

一方面，序列号可以判断报文的时效性，防止旧连接的历史报文被新连接接收了。另一方面，因为序列号是随机生成的，不容易伪造，可以一定程度上保证安全性。

初始序列号 ISN 的随机产生算法

一开始的 ISN ⽣成算法是基于时钟的，每 4 毫秒 + 1，转一圈 4.55 个小时。RFC 1948 中提出了⼀个更好的 ISN 随机⽣成算法。

$ISN = M + F (localhost, localport, remotehost, remoteport)$

- M 是⼀个计时器，这个计时器每隔 4 毫秒 +1。 
- F 是⼀个 Hash 算法，根据源 IP、源端口、目标 IP、目标端口生成⼀个随机数值。

两个不同的报文的序列号是有可能一样的。因为 seq 的计算方式是 $seq = seq + 数据长度$。 比如，第三次握手，发送端如果没有携带数据，这个时候的 seq num 就和第一次握手的 seq num 是一样的。

#### 确认应答号

确认应答号用于解决数据包丢包的问题。

确认应答号是下一次期望收到的数据包的序列号。发送端收到确认应答号后可以认为，在这个序列号以前的数据都已被接收端正常接收。如果下一个期望收到的数据包没有收到，那么确认应答号就不会发生变化。

确认应答号存在累积确认机制。当接收端收到多个数据包时，只需要应答最后一个包的 ACK 报文就可以了。当接收端收到比期望序列号大的报文时，会重复应答最近一次的应答报文的确认应答号。

#### 控制位

- ACK：ACK 为 1 时，确认应答号字段变为有效。
TCP 规定，除了最初建立连接时的 SYN 包之外，该值必须为 1。
- RST：RST 为 1 时，表示 TCP 连接出现异常，必须强制断开连接。
- SYN（Synchronize Sequence Numbers、同步序列号）：SYN 为 1 时，表示希望建立连接，同时序列号字段会设置初始值。
- FIN：FIN 为 1 时，表示希望断开连接。

#### 数据长度

$TCP 数据长度 = IP 数据包总长度 - IP 头部长度 - TCP 头部长度$

报文的数据段过大时，按 MSS（TCP 最大报文长度）分成 TCP Segment（TCP 块）。MSS 的值，在建立连接时，由双方协商得出。 

让 TCP 处理分块，避免 IP 层分片，可以提高传输性能。IP 没有超时重传机制，超时重传机制由 TCP 负责。如果让 IP 来分片，那么其中任意一个 IP 分片有问题，都要重传整个 IP 报文。TCP 分块后，当中间某个分块有问题时，只需要重传这个分块。

### 参考

- [https://www.rfc-editor.org/rfc/rfc793](https://www.rfc-editor.org/rfc/rfc793)
- [小林coding](https://xiaolincoding.com/)
  - [图解网络](https://xiaolincoding.com/network/)