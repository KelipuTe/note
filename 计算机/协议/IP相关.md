---
draft: true
date: 2022-05-04 08:00:00 +0800
title: "和 IP 相关"
summary: "和 IP 相关的玩意，暂时放这里"
toc: true

categories:
- protocol(协议)

tags:
- computer-science(计算机科学)
- protocol(协议)
- ip
---

### DNS

DNS（Domain Name System、域名系统）是一个将域名和 IP 地址相互映射的分布式数据库。作用是把服务端域名解析到服务端的 IP 地址。DNS 服务主要是基于 UDP 实现的，服务器的端口号为 53。

#### 域名的层级关系

DNS 中的域名是用句点（.）来分隔的，越靠右的位置表示其层级越高。整个域名的层级关系类似一个树状结构。

这里以百度的域名为例：www.baidu.com。最顶层是根域名，下一层是顶级域名（com），再下一层是权威域名（baidu.com）。

所有的 DNS 服务器都有根服务器的信息。所以，只要连上任意一台 DNS 服务器，就可以找到根服务器。然后，顺着树状结构就可以找到目标域名。

#### 根域名

根域名是由一群服务器组成的，这群服务器用了 13 个不同的域名。从 a~m 开头，后面都是 `.root-servers.net`，如 a.root-servers.net。

13 个根域名由上千台根服务器（节点）组成，由 12 个独立机构运营。也就是说，绝大部分的根域名的节点的 IP 地址是一样的。通过任波技术，当向根服务器进行联系时，就能找到最近的根域名服务器。

- 根域名服务器会管理 .com、.cn、.net 这些 TLD（Top Level Domain、顶级域名）服务器。
- 顶级域名服务器会管理各自的 Authoritative（权限域名）服务器也叫权威域名服务器。
- 权限域名服务器会管理各自的主机服务器。

#### 域名结构树

这里以 bilibili 的两个域名为例：www.bilibili.com 和 space.bilibili.com。www.bilibili.com 表示访问 bilibili 的主站。space.bilibili.com 表示访问 bilibili 的个人空间。

- .bilibili 是权威域名。www.bilibili.com 和 space.bilibili.com 都被 .bilibili 域名管理。
- .com 是顶级域名。.bilibili 域名被 .com 域名管理。
- .com 域名又是被根域名（.）管理的。但是，一般省略根域名。完整的域名应该是 www.bilibili.com.，后面还有个句点表示根域名，也就是 .root 。

#### 域名解析的工作流程

这里以浏览器为例。

第一步、浏览器会搜索浏览器的 DNS 缓存。

如果浏览器的 DNS 缓存中没有对应的域名，那么会搜索操作系统的 DNS 缓存。如果操作系统的 DNS 缓存中没有对应的域名，那么会搜索操作系统的 hosts 文件。如果 host 文件中没有对应的域名，那么进入第二步。

第二步、浏览器通过"客户端 DNS 解析器"，向"本地 DNS 服务器"发起域名解析请求。

本地 DNS 服务器一般由互联网服务提供商（ISP）负责管理，比如中国电信。客户端机器的操作系统的 TCP/IP 设置中，会配置一个本地 DNS 服务器。最后本地 DNS 服务器会将域名对应的 IP 地址返回给客户端。

客户端 DNS 解析器与本地 DNS 服务器之间使用递归方式传输。客户端 DNS 解析器发起请求之后，就等着本地 DNS 服务器返回结果。

#### 本地 DNS 服务器的工作流程

第一步、本地 DNS 服务器会查询自己的缓存。如果缓存里能找到就直接返回。

第二步、如果缓存里找不到，本地 DNS 服务器就会访问根服务器，请求查询 www.baidu.com。www.baidu.com 域名归 com 顶级域管理。所以，根服务器会返回顶级域服务器的地址。

本地 DNS 服务器在得到根服务器返回的地址后，继续访问顶级域服务器，请求查询 www.baidu.com。www.baidu.com 域名归 baidu.com 权威域管理。所以，顶级域服务器会返回权威域服务器的地址。

本地 DNS 服务器在得到顶级域服务器给的地址后，继续访问权威域服务器，请求查询 www.baidu.com。权威域服务器查询到结果后，将 www.baidu.com 对应的 IP 地址返回给本地 DNS 服务器。

本地 DNS 服务器和其他域名服务器之间使用迭代方式传输。本地 DNS 服务器会依次访问其他的域名服务器，最终得到 IP。

### ARP

ARP（Address Resolution Protocol、地址解析协议）的作用是根据 IP 地址获取 MAC 地址（物理地址）。

主机会以广播的形式发送 ARP 请求，这个请求中包含目标主机的 IP 地址。

同一个链路中的所有设备在收到 ARP 请求时都会拆开 ARP 请求包。如果请求包中的目标 IP 和自己的 IP 地址一致，就会将自己的 MAC 地址通过 ARP 响应包返回给主机。

主机在获取到 MAC 地址后，会将其缓存起来，下次会直接从缓存中获取。

### RARP

RARP（Reverse Address Resolution Protocol、反向地址转换协议）的作用是根据已知的 MAC 地址获取 IP 地址。

RARP 通常需要架设一台服务器，在这个服务器上注册设备的 MAC 地址及其 IP 地址。其它设备向 RARP 服务器发送 RARP 请求包，然后 RARP 服务器通过 RARP 响应包返回 IP 地址。

### DHCP

DHCP（Dynamic HostConfigurationProtocol、动态主机配置协议）是一个应用层的局域网的网络协议。通过 DHCP 动态获取 IP 地址，省去了配置 IP 地址的过程。由服务器控制一段范围的 IP 地址，客户机登录服务器时就可以自动获得服务器分配的 IP 地址和子网掩码。

DHCP 全程使用 UDP 广播通信。客户端进程监听 68 端口，服务端进程监听 67 端口。

第一步、客户端发送 DHCP DISCOVER（DHCP 发现报文）的 IP 数据报。

由于客户端这个时候没有 IP 地址，也不知道 DHCP 服务端的 IP 地址，所以使用 UDP 广播进行通信。目标 IP 地址设置为广播地址 255.255.255.255:67，源 IP 地址设置为 0.0.0.0:68。报文主要携带一个事务 ID。

第二步、DHCP 服务端收到 DHCP 发现报文时，用 DHCP OFFER（DHCP 提供报文）响应客户端。

报文携带事务 ID、DHCP 服务器地址、可供租约的 IP 地址、子网掩码、默认网关、DNS 服务器、IP 地址租用期。使用 UDP 广播进行通信。目标 IP 地址设置为 255.255.255.255:68，源 IP 地址设置为自己的地址。

第三步、客户端收到一个获多个服务器提供的 DHCP 提供报文后，从中选择一个。然后向选中的服务器发送 DHCP REQUEST（DHCP 请求报文）进行响应。

报文携带事务 ID、DHCP 服务器地址、回传配置信息（租约的 IP 地址、租期等）。目标 IP 地址设置为 255.255.255.255:67，源 IP 地址设置为 0.0.0.0:68。

第四步、服务端收到 DHCP 请求报文后，用 DHCP ACK 响应客户端。

报文携带事务 ID、DHCP 服务器地址、确认配置信息（租约的 IP 地址、租期等）目标 IP 地址设置为 255.255.255.255:68，源 IP 地址设置为自己的地址。

当客户端和 DHCP 服务器不在一个局域网中时，由于路由器不会转发广播包，所以需要其他设备进行处理。这个设备就是 DHCP relay agent（中继代理），通过 DHCP 中继代理，一个 DHCP 服务器就可以对不同的网段进行 IP 地址分配。

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/protocol/ip/dhcp_relay_agent.drawio.png">)

[//]: # (</div>)

当续约的 IP 地址到期时，客户端会向服务器发送 DHCP 请求报文。

- 如果服务器同意继续租用，则用 DHCP ACK 进行应答。客户端可以延长租期继续使用 IP 地址。
- 如果服务器不同意继续租用，则用 DHCP NACK 进行应答。客户端就要停止使用 IP 地址。

### NAT 和 NAPT

简单地说 NAT（Network Address Translation、网络地址转换）就是同一组织、公司内的主机对外通信时，把私有 IP 转换成公有 IP。

转换方式有：

- Static Nat（静态转换）
- Dynamic Nat（动态转换）
- Over Load（端口多路复用）。

NAPT（Network Address Port Translation、网络地址端口转换）

由于绝大多数的网络应用都是使用传输层的 TCP 或者 UDP。因此可以把内部的私有 IP转换成公有 IP + 端口的形式，然后生成一个 NAPT 路由转换表。路由器根据路由转换表，将从互联网收到的数据包分发给内网指定的机器。

NAT 和 NAPT 都需要依赖路由转换表。

- 1、这意味着外部无法主动与内部机器建立连接。
- 2、维护转换表和进行转换操作都需要额外的性能开销。
- 3、一旦路由器重启，所有的 TCP 连接都会被重置。

解决方案有两种：用 IPv6 或者 NAT 穿透技术。

NAT 穿透技术，能够让应用程序主动发现自己位于 NAT 设备之后。程序会主动获得 NAT 设备的公有 IP 并为自己建立端口映射。然后用这个映射对外通信。

### ICMP

ICMP（Internet Control Message Protocol、互联网控制报文协议）是 TCP/IP 协议簇的一个子协议，用于在 IP 主机、路由器之间传递控制消息。控制消息是指网络通不通、主机是否可达、路由是否可用等网络本身的消息。

ICMP 主要的功能包括：

- 确认 IP 包是否成功送达目标地址
- 报告发送过程中 IP 包被废弃的原因和改善网络设置等。

在 IP 通信中如果某个 IP 包因为某种原因未能达到目标地址，那么这个具体的原因将由 ICMP 负责通知。

ICMP 大致可以分为两大类：

- 查询报文类型，用于诊断的查询消息
- 差错报文类型，用于通知出错原因的错误消息。

### IGMP

主机与本地路由器之间使用 IGMP（Internet Group Management Protocol、因特网组管理协议）来进行组播组成员信息的交互，管理组播组成员的加入和离开。

主机 IP 软件需要进行组播扩展，才能使主机能够在本地网络上收发组播分组。但仅靠这一点是不够的，因为跨越多个网络的组播转发必须依赖于路由器。路由器为建立组播转发路由必需了解每个组员在 Internet 中的分布。这要求主机必须能将其所在的组播组通知给本地路由器。

### 参考

- [小林coding](https://xiaolincoding.com/)
  - [图解网络](https://xiaolincoding.com/network/)
- [DNS解析的过程是什么，求详细的？](https://www.zhihu.com/question/23042131/answer/1683528809)