---
draft: true
title: "IP 协议"
summary: "IP 协议；"
toc: true

categories:
  - 协议

tags:
  - 计算机
  - 协议

date: 2022-05-04 08:00:00 +0800
---

## 反向链接

## 资料

## 正文

- IP：Internet Protocol、网际互连协议

### IP

IP 根据端到端的设计原则，
为主机提供一种无连接的、不可靠的、尽力而为的数据包传输服务。

### 网络层

网络层的主要作用是实现主机与主机之间点对点的通信。网络层的通信只有两个主体，即两个主机。

网络层负责在没有实现直连的两个网络之间进行通信，数据链路层负责在实现直连的两个设备之间进行通信。
即两个主机之间通过多个网络基础设施节点实现互联，数据链路层负责这些网络基础设施节点两两之间的通信。

在传输过程中源 IP 地址和目标 IP 地址在是不会变化的但是 MAC 地址，每经过一个基础设施节点就会变一次。

### IPv4

IPv4（Internet Protocol version 4、互联网通信协议第四版）
是由 IETF（The Internet Engineering Task Force、国际互联网工程任务组） 的 RFC 791 定义的。

[RFC 791](https://www.rfc-editor.org/rfc/rfc791)；
[RFC 791](https://datatracker.ietf.org/doc/html/rfc791)；

#### IPv4 地址

```
High Order Bits   Format                           Class
---------------   -------------------------------  -----
      0            7 bits of net, 24 bits of host    a
      10          14 bits of net, 16 bits of host    b
      110         21 bits of net,  8 bits of host    c
      111         escape to extended addressing mode
```

IPv4 地址由 32 位二进制数表示，分为网络号和主机号。最多可以表示 $2^{32}$ 个主机。

##### 网络号

IP 地址的网络号用于路由控制。

两台主机需要通信，首先需要判断在不在同一个网络内，即网络号是否一样。

- 如果网络号相同，那说明双方在同一个网络内，可以把数据包直接发过去。
- 如果网络号不同，那说明双方不在同一个网络内，这时就需要路由器进行转发。

路由器会维护一个路由表。路由表中记录着网络号对应的下一跳应该发送的地址。路由器在转发数据包时，首先从 IP
报文头部中获取目标地址，计算出网络号。然后从路由表中找到与该目标地址具有相同网络号的记录，将数据包转发给下一跳的地址。

如果路由表中存在多条相同网络号的记录，就选择与目标地址相同位数最多的记录。

##### 子网掩码

子网掩码可以用来分离网络号和主机号。掩码的意思就是掩盖掉主机号，剩余的就是网络号。如 IP 地址 10.100.122.2，子网掩码
255.255.255.0。用 IP 地址和子网掩码作按位与运算，就可以得到网络号 10.100.122.0。

子网掩码的另一个作用是划分子网。子网划分将主机号再分为，子网网络号和子网主机号两部分。如 IP 地址 192.168.1.0，设置子网掩码为
255.255.255.192。可以将 8 位主机号分为 2 位子网网络号和 6 位子网主机号。

##### 点分十进制

IPv4 地址可以写作点分十进制的形式。将 32 位二进制数，每 8 位分成一组，并转换为 10 进制数，中间用小数点连接。如地址 11000000
10101000 00000001 00000001 可写作 192.168.1.1。

各类 IPv4 地址的范围：

- A 类：0.0.0.0 ~ 127.255.255.255；
- B 类：128.0.0.0 ~ 191.255.255.255；
- C 类：192.0.0.0 ~ 223.255.255.255；
- D 类：224.0.0.0 ~ 239.255.255.255；
- E 类：240.0.0.0 ~ 255.255.255.255；

#### 主机号全为 0 或 1

对于 A、B、C 类地址，它们都有两个特殊的地址。

- 主机号全为 0，指定某个网络，通常用在路由表中。
- 主机号全为 1，指定某个网络下的所有主机，通常用于广播。

所以对于 A、B、C 类地址，可连接的最大主机数需要减 2。

广播就是向广播地址指定的网络中所有的主机发送数据包。广播地址可以分为local broadcast（本地广播）和 directed
broadcast（直接广播）两种。

在本网络内广播的叫本地广播。比如主机 192.168.0.1 向 192.168.0.0/24 的广播地址 192.168.0.255 发送数据包。192.168.0.0/24
网络下的所有主机 192.168.0.2 ~ 192.168.0.254 都能收到数据包，而 192.168.0.0/24 之外的网络不会。

在不同网络之间广播的叫直接广播，比如主机 192.168.0.1/24 向 192.168.1.0/24 的广播地址 192.168.1.255
发送数据包。由于发送主机和目标广播地址不在一个网络中，所以数据包需要路由器进行转发。然后 192.168.1.0/24 网络下的所有主机
192.168.1.1 ~ 192.168.1.254 都能收到数据包。

#### 公有地址和私有地址

对于 A、B、C 类地址，它们都有公有地址和私有地址的概念。

公有地址必须唯一，私有地址可以重复。比如两个公司各自的服务器的公有地址必须不一样，这样客户端才知道该访问哪台服务器。私有地址通常只在公司或组织内部使用，两个公司内部有私有地址相同并不会造成冲突。

公有地址由 ICANN（互联网名称与数字地址分配机构）管理。私有地址由各个公司或组织内部的 IT 人员自行管理。ICANN 下属的 IANA
负责按大洲，层层分配地址。中国地区的公有地址是由 IANA 下属的 APNIC 下属的 CNNIC 管理。

私有地址的范围：

- A 类：10.0.0.0 ~ 10.255.255.255；
- B 类：172.16.0.0 ~ 172.32.255.255；
- C 类：192.168.0.0 ~ 192.168.255.255；

#### D、E 类地址

D、E 类地址没有主机号。D 类地址常用于组播（多播），E 类地址暂未使用。

组播用于将数据包发给特定组内的所有主机。广播无法穿透路由，广播只能给某一个网络下的所有主机发送数据包。如果想给处于不同网络中的一组主机发送同样的数据包就可以使用组播。

224.0.0.0 ~ 239.255.255.255 都是组播的可用范围，其划分为以下三类。

- 224.0.0.0 ~ 224.0.0.255 为预留的组播地址，只能在局域网中使用，路由器不会进行转发。
- 224.0.1.0 ~ 238.255.255.255 为用户可用的组播地址，可用于 Internet。
- 239.0.0.0 ~ 239.255.255.255 为本地管理组播地址，可用于内网，仅在特定的本地范围内有效。

#### unicast（单播）、broadcast（广播）、multicast（组播）

#### 回环地址

回环地址是在同一台计算机上的程序之间进行网络通信时所使用的一个默认地址。

计算机使用 127.0.0.1 作为回环地址。当使用回环地址时，传输的数据包不会流向互联网。

主机名 localhost 与回环地址127.0.0.1 有相同意义。

#### CIDR（Classless Inter-Domain Routing、无分类地址）

无分类地址不再有分类的概念，直接将 32 位 二进制数分为两部分。

前面是网络号，后面是主机号。如 192.168.0.0/24 ，表示网络号占 24 位。

#### IPv4 报文结构

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version|  IHL  |Type of Service|          Total Length         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |    Protocol   |         Header Checksum       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             data                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- Version：版本号，4 bit
- IHL：Internet Header Length、头部长度，4 bit
- TOS：Type of Service、服务类型，8 bit
- Total Length：总长度，16 bit
- Identification：标识符，16 bit
- Flags：标志，3 bit；
- Fragment Offset：分片偏移，13 bit
- TTL：Time To Live、生存时间，8 bit
- Protocol：协议，8 bit
- Header Checksum：头部校验和，16 bit
- Source Address：源地址，32 bit
- Destination Address：目标地址，32 bit
- Option、Padding：选项和填充，长度可变
- Data：数据，长度可变

为了网络设备硬件设计和处理方便，首部长度需要是 4 字节的整数倍。

每种数据链路的最大传输单元（MTU）都是不同的，比如以太网的 MTU 是 1500 字节。
当 IP 数据包的大小大于 MTU 时，数据包就会被分片。经过分片后的 IP 数据报只能由目标主机进行重组。

在进行分片传输时，一旦某个分片丢失，那么整个 IP 数据包就无效了，需要全部重传。
TCP 引入了 MSS，数据在 TCP 层进行分包，不交给 IP 层分片。UDP 尽量不要发送超过 MTU 的数据包。

### IPv6

IPv6（Internet Protocol version 6、互联网通信协议第 6 版）由 IETF（The Internet Engineering Task Force、国际互联网工程任务组）
的 RFC 2460 定义。

- [https://www.rfc-editor.org/rfc/rfc2460](https://www.rfc-editor.org/rfc/rfc2460)
- [https://datatracker.ietf.org/doc/html/rfc2460](https://datatracker.ietf.org/doc/html/rfc2460)

#### IPv6 地址

IPv6 地址由 128 位二进制数表示。最多可以表示 $2^{128}$ 个主机。

IPv6 地址可以写作 16 进制的形式。将 128 位二进制数，每 16 位分成一组，并转换为 16
进制数，中间用冒号连接。二进制：`16 位 : 16 位 : 16 位 : 16 位 : 16 位 : 16 位 : 16 位 : 16 位`
。可写作：`4 位 : 4 位 : 4 位 : 4 位 : 4 位 : 4 位 : 4 位 : 4 位`。

连续的 0 可以用两个冒号表示，但是每个地址中只能出现一次。如：`4 位 : 0000 : 0000 : 0000 : 0000 : 0000 : 0000 : 4 位`
可写作 `4 位 :: 4 位`

- IPv6 没有广播地址。
- 单播地址用于一对一通信。
- 组播地址用于一对多通信。
- 任播地址用于通信最近的节点，最近的节点由路由协议决定。

单播地址主要分为三类：

- 在同一链路里，不经过路由器，使用链路本地单播地址。
- 在内网里，使用唯一本地地址，相当于 IPv4 的私有 IP。
- 在互联网上，使用全局单播地址，相当于 IPv4 的公有 IP。

#### IPv6 报文结构

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version| Traffic Class |           Flow Label                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Payload Length        |  Next Header  |   Hop Limit   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                         Source Address                        +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                      Destination Address                      +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- Version：版本号，4 bit
- Traffic Class：流量类型，8 bit
- Flow Label：流量标记，20 bit
- Payload Length：有效载荷长度，16 bit
- Next Header：下一个报头，8 bit
- Hot limit：跳数限制，8 bit
- Source Address：源地址，128 bit
- Destination Address：目标地址，128 bit

$Payload Length = 报头后的数据长度 + 扩展头部数据长度$

扩展头部可以有多个。IPv6 头部的 Next Header 存储第一个扩展头部的位置。第一个扩展头部的 Next Header 存储的是第二个扩展头部的位置。

- IPv6 取消了首部校验和字段。IPv4 中这个字段在数据链路层和传输层都会校验，IPv6 直接取消校验。
- IPv6 取消了分片和重组相关字段。分片和重组很耗时，IPv6 不允许中间路由器进行分片与重组。
- IPv6 取消选项字段。IPv6 中选项字段并没有消失，而是放在扩展头部中。

#### IPv6 相比 IPv4 的优点

- IPv6 首部长度固定为 40 个字节，去掉了包头校验，简化了首部结构，从而减轻路由器负荷，提高传输性能。
- IPv6 不需要 DHCP 服务器也可以实现自动分配 IP 地址。
- IPv6 有应对伪造 IP 地址的网络安全功能以及防止线路窃听的功能，提高了安全性。
- ...
