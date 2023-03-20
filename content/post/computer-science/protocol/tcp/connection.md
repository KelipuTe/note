---
draft: false
date: 2022-05-02 08:00:00 +0800
lastmod: 2022-05-02 08:00:00 +0800
title: "TCP 连接"
summary: "TCP 连接"
toc: true

categories:
- protocol(协议)

tags:
- computer-science(计算机科学)
- protocol(协议)
- tcp
---

### 连接

连接一定是一对一的、唯一的。连接的唯一性由 TCP 四元组（源地址、源端口号、目标地址、目标端口号）保证。源地址、目标地址在 IP 报文中，源端口号、目标端口号在 TCP 报文中。

建立一个 TCP 连接需要三个信息的共识：由 IP 地址和端口号组成的 Socket、序列号、窗口大小。

#### 连接数

$最大连接数 = 端 IP 数 \times 端端口数$

对于 IPv4 来说：$端 IP 数 = 2^{32}$、$端端口数 = 2^{16}$。但是，端最大并发连接数远不能达到理论上限。

- 1、socket 文件也是文件，受系统对文件描述符的限制。
- 2、每个连接都需要消耗内存，而系统内存有限。

#### 保活机制

在一个时间段内，如果没有任何连接相关的活动，TCP 保活机制就会开始作用。每隔一段时间，发送一个探测报文，如果连续几个探测报文没有得到应答，则认为当前连接已死亡。

在 Linux 内核中可以设置，保活时间、保活探测次数、保活探测时间间隔三个参数。

- net.ipv4.tcp_keepalive_time，默认保活时间是 7200 秒。
- net.ipv4.tcp_keepalive_intvl ，默认每次检测间隔 75 秒。
- net.ipv4.tcp_keepalive_probes，默认检测 9 次无响应，就认为连接死亡。

也就是最少也要经过 7875 秒，系统才能发现一个死亡的连接。

#### Fast Open

客户端在向服务端发起 HTTP GET 请求时，一个完整的交互需要 2.5 个 RTT 时延。如果算上第三次握手可以携带数据，这时也需要 2 个 RTT 时延。但是当下一次发起请求时，又需要再来一次。

Linux 3.7 提供了 TCP Fast Open 功能，可以减少 TCP 连接建立时的时延。

第一次建立连接的时候，服务端在第二次握手的时候产生一个加密的 Cookie ，和 ACK + SYN 报文一起返回。

客户端可以缓存这个 Cookie，下一次发起请求的时候，在第一次握手时，直接发送 SYN + Cookie + HTTP GET。

这样服务端直接从 Cookie 中获取相关信息，就可以跳过三次握手的过程。这时 HTTP GET 请求就只需要 1 个 RTT。

客户端可以一直用这个 Cookie，直到服务端认为 Cookie 无效（通常是过期了）。

在 Linux 中，通过设置 `/proc/sys/net/ipv4/tcp_fastopen` 来操作 TCP Fast Open。一共有 4 个值：0（关闭）；1（作为发起端使用）；2（作为响应端使用）；3（作为任意一端使用）。

#### 查看 TCP 连接状态

在 Linux 中：

- `netstat -napt`：查看 TCP 连接状态
- `ss`：查看 TCP 全连接队列的状态

在 Windows 中，可以使用软件 Wireshark 抓包。

### 三次握手

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/protocol/tcp/three_handshake.drawio.png">)

[//]: # (</div>)

初始时，发起端和响应端都处于关闭状态。响应端开始主动监听己方系统的某个端口号是建立连接的前提。

#### 第一次握手

发起端发送 SYN 报文，控制位 SYN 设置为 1，序列号初始化为 client_isn。

##### 第一次握手一直丢包

发起端会进行超时重传，在重复 5 次发送 SYN 报文之后断开。每次重传间隔时间为：1、3、7、15、31，符合 RTO 翻倍的逻辑。

在 Linux 中，第一次握手的超时重传的次数可以通过 `/proc/sys/net/ipv4/tcp_syn_retries` 设置。默认是 5 次。

##### 第一次握手的优化

优化发起端第一次握手的超时重试的次数（`/proc/sys/net/ipv4/tcp_syn_retries`），直接给应用程序报错。

#### 第二次握手

响应端应答 ACK 报文，控制位 ACK 设置为 1，确认应答号设置为 client_isn + 1。同时发送己方的 SYN 报文，控制位 SYN 设置为 1，序列号初始化为 server_isn。

##### 第二次握手一直丢包

第一种情况：响应端没收到第三次握手的 ACK 报文。

响应端会进行超时重传，重传第二次握手的 ACK + SYN 报文，次数不定。

在 Linux 中，第二次握手的超时重传次数可以通过 `/proc/sys/net/ipv4/tcp_synack_retries` 设置。默认是 5 次。

第二种情况：发起端没收到第二次握手的 ACK + SYN 报文。

发起端会进行超时重传，重传第一次握手的 SYN 报文。

响应端收到 SYN 报文，应答第二次握手的 ACK + SYN 报文，然后重新设定超时重传的计时器。

最后，发起端在重复 5 次发送 SYN 之后断开。

##### 第二次握手的优化

1、增加半连接队列和全连接队列的长度。

注意：只增加半连接队列是无效的。

- `/proc/sys/net/ipv4/tcp_max_syn_backlog`、
- `/proc/sys/net/core/somaxconn`、
- 系统函数 listen（[listen(2) - listen for connections on a socket](https://man7.org/linux/man-pages/man2/listen.2.html)） 的 backlog 参数。

2、开启 cookie，`/proc/sys/net/ipv4/tcp_syncookies`。

3、优化响应端第二次握手的超时重试的次数（`/proc/sys/net/ipv4/tcp_synack_retries`），直接给应用程序报错。

#### 第三次握手

发起端应答 ACK 报文，控制位 ACK 设置为 1，确认应答号设置为 server_isn + 1。

第三次握手的时候就可以携带数据了，前两次是不可以的。

##### 第三次握手一直丢包

响应端会进行第二次握手的超时重传，在重复 5 次发送 ACK + SYN 报文之后断开。

响应端断开后，发起端依然处于 established 的状态，不会断开。

这个时候如果发起端发送报文，就会一直重传，在 Linux 中，默认重传 15 次后停止。这个属于 TCP 建立连接后的重传次数。在 Linux 中，可以通过 `/proc/sys/net/ipv4/tcp_retries2` 设置。

最终由 TCP 的保活机制决定发起端什么时候断开连接。

#### 为什么需要三次握手

1、同步双方的序列号

发起端和响应端，各自发送并确认己方序列号的逻辑。如果是独立的过程，最多需要两个来回也就是四次握手。但是响应端的答可以和问同时进行，所以可以合并为三次。

2、防止历史连接和资源浪费

发起端由于某种原因发起一次连接后关闭，然后又发起一次。这时前一次连接的无效的 SYN 报文依然可能到达响应端。但是响应端并不知道这两个报文的先后顺序，所以都会应答 ACK + SYN 报文。这时响应端会初始化两个连接。

发起端接到两个应答报文后，可以通过自己的上下文判断，是历史连接还是新的连接。如果是历史连接，发起端需要发送 RST 报文告知响应端断开历史连接。如果是新的连接，发起端就进行第三次握手建立连接。如果没有第三次握手，响应端两个建立到一半的连接就无法取舍，从而造成资源浪费。

### 三次握手在内核中的流程

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/protocol/tcp/three_handshake_linux_core.drawio.png">)

[//]: # (</div>)

#### 半连接队列和全连接队列

响应端收到发起端的 SYN 之后，内核会把连接存储到半连接队列（SYN 队列）中，然后响应 ACK + SYN。

当响应端接收到发起端第三次握手的 ACK 报文之后，内核会把连接从半连接队列中移除，
然后，内核会创建新的连接并添加到全连接队列（Accept 队列），等待应用程序调用系统函数 accept（[accept(2) - accept a connection on a socket](https://man7.org/linux/man-pages/man2/accept.2.html)） 把连接取走。

不管是半连接队列还是全连接队列都有队列长度，超过限制时会直接丢弃或者返回 RST 报文。

半连接队列的最大值，取决于三个参数：

- `net.ipv4.tcp_max_syn_backlog`
- `/proc/sys/net/core/somaxconn`
- 系统函数 listen 的 backlog 参数。

- 当 `max_syn_backlog > min(somaxconn, backlog)` 时， 半连接队列最⼤值为 `min(somaxconn,backlog) * 2`。
- 当 `max_syn_backlog < min(somaxconn, backlog)` 时， 半连接队列最⼤值为 `max_syn_backlog * 2`。

全连接队列的长度取决于两个参数，全连接队列的大小是两个参数值的最小值。

- `/proc/sys/net/core/somaxconn`，Linux 默认 128。
- 系统函数 listen 的 backlog 参数，Nginx 默认 511。

`/proc/sys/net/ipv4/tcp_abort_on_overflow`，控制全连接队列满时的动作。

- `tcp_abort_on_overflow = 0`，全连接队列满时，丢弃 ACK。Linux 默认 0。
- `tcp_abort_on_overflow = 1`，全连接队列满时，发送 RST 包，断掉连接。

对于突发流量，应该设置 `tcp_abort_on_overflow = 0`，让客户端重发。如果持续不断有连接被丢弃，也不应该设置 1，而是应该修改全连接队列的大小。

丢弃连接的条件：

- 1、半连接队列满了，且没有开启 tcp_syncookies。
- 2、全连接队列满了，且没有重传 SYN + ACK 包的连接请求大于 1.
- 3、没有开启 tcp_syncookies，且 `max_syn_backlog - 当前半连接队列长度 < (max_syn_backlog >> 2)`

#### SYN 攻击

SYN 攻击与 SYN 洪泛、DDos 攻击的思路差不多。攻击者同一时间向服务端发送大量伪造的 IP 地址不同的 SYN 报文。大量没有得到应答的连接最终会占满服务端的 SYN 队列，使得不能为正常用户提供服务。

解决方案 1：修改 Linux 参数，控制队列大小和队列满时的操作。

- net.core.netdev_max_backlog，队列最大值。
- net.ipv4.tcp_max_syn_backlog，连接处于 syn_rcvd 状态的最大值。
- net.ipv4.tcp_abort_on_overflow，队列满时的操作。

解决方案 2：开启 cookie。

- `net.ipv4.tcp_syncookies = 1`。

开启后，当 SYN 队列满时，后续的连接不进入 SYN 队列。响应端会计算一个 cookie，放在 ACK + SYN 报文的序列号中返回。当接收到 ACK 时，验证合法性，如果合法，就放入 Accept 队列。

解决方案 3：减少 ACK + SYN 重传的次数（/proc/sys/net/ipv4/tcp_synack_retries）。

### 关闭连接

关闭连接的方式有两种，FIN 报文和 RST 报文。

如果进程异常退出，Linux 内核就会发送 RST 报文给对方，这个过程不走四次挥手。

想要安全关闭连接必须通过四次挥手，可以通过系统函数 close（[close(2) - close a file descriptor](https://man7.org/linux/man-pages/man2/close.2.html)） 和系统函数 shutdown（[shutdown(2) - shut down part of a full-duplex connection](https://man7.org/linux/man-pages/man2/shutdown.2.html)）发起。

调用 close 意味者完全断开连接，进程无法接收和发送数据。调用方这边也叫孤儿连接。shutdown 可以关闭一个方向的连接，可以只关闭读，或者只关闭写，也可以全部关闭。

### 四次挥手

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/protocol/tcp/four_wave.drawio.png">)

[//]: # (</div>)

#### 第一次挥手

发起端发送 FIN 报文，控制位 FIN 设置为 1。

##### 第一次挥手一直丢包

Linux 内核会定时重发 FIN 报文。超过重传次数后会直接关闭连接。

重传次数由 `/proc/sys/net/ipv4/tcp_orphan_retires` 控制。参数默认值为 0，但是含义是重传 8 次。这个参数不止对孤儿连接有效，还对所有处于 fin_wait_1 状态的连接有效。

遭到恶意攻击时，FIN 报文有可能发不出去。

- 1、TCP 必须保证报文是有序的，FIN 报文也不例外。如果发送缓冲区还有数据没发送时，FIN 报文是不能提前发送的。
- 2、TCP 的流量控制，当接收方窗口为 0 时，发送方就不再发送数据。

攻击场景：攻击方请求下载大文件，然后将己方的接收窗口设为 0。这样服务端的 FIN 报文发不出去，就会有大量的连接卡在 fin_wait_1 状态。

这种情况可以调整 Linux 系统的 `/proc/sys/net/ipv4/tcp_max_orphans` 参数，控制最大孤儿连接数量。防止孤儿连接过多，长时间占用系统资源。这时新增的孤儿连接将不再走四次挥手的流程，而是直接发送 RST 报文强制关闭。

#### 第二次挥手

响应端应答发起端 ACK 报文。然后进入 closed_wait 状态。也就是等待程序调用 close 关闭连接。如果响应端发现有大量的连接处于 closed_wait 状态，那大概率是程序有 bug，没有及时调用 close。

响应端处理完现有的报文，然后就会进入第三次挥手的阶段。

#### 第二次挥手的优化

发起端接收到响应端的 ACK 报文后进入 fin_wait_2 状态。这个状态表示发起端的发送通道已关闭，等待响应端发送 FIN 报文。

- 对于 shutdown 关闭的连接，可以一直处于 fin_wait_2 状态。
- 对于 close 关闭的连接，由于无法再接收数据，所以 fin_wait_2 状态不能持续太久。`/proc/sys/net/ipv4/tcp_fin_timeout` 控制这个状态下连接的持续时长，默认是 60 秒，和 time_wait 是一样的。

孤儿连接，在 60 秒内没有收到 FIN 报文，超时后连接就会直接关闭。

#### 第三次挥手

响应端发送 FIN 报文，控制位 FIN 设置为 1。

如果响应端没有待处理的报文，那么第二次挥手和第三次挥手的报文可能是一起发送的。

#### 第四次挥手

发起端应答响应端 ACK 报文。然后，进入 time_wait 状态，经过 2 MSL 之后，自动关闭。这里发起端指的是主动关闭的那端，主动关闭的那端才会有 time_wait 状态。响应端收到 ACK 报文后，关闭。

##### 第三次挥手或者第四次挥手一直丢包

对于响应端来说，都是一直没有收到 ACK 报文。这时 Linux 内核会定时重发 FIN 报文。
重传次数依然由 `/proc/sys/net/ipv4/tcp_orphan_retires` 参数控制。

#### 为什么要有 time_wait 状态

time_wait 状态主要有两个作用：

- 1、防止有相同四元组的旧数据包被接收。

如果没有 time_wait，相同端口的 TCP 连接被复用时，可能会收到上次连接中没有及时到达的被延迟的数据包。2 MSL 的时间，足够让两个方向上的数据包都被丢弃，这样后续的数据包一定都是新的连接产生的。

- 2、保证被动关闭连接的一方能被正确关闭。

如果没有 time_wait，响应端又没收到 ACK 报文时，响应端就会一直卡在 last_ack 状态。
这时发起端使用相同端口建立新的连接时，响应端由于卡在 last_ack 状态，所以会直接返回 RST 终止连接。

#### 为什么 time_wait 等待 2 MSL

MSL（Maximum Segment Lifetime、报文最大生存时间）：指的是任何报文在网络上存在的最大时间。

如果响应端没有接到最后的 ACK 报文，就会重发 FIN 报文。发起端应答 ACK 报文和响应端重发 FIN 报文，一来一回是 2 MSL。

在 Linux 中，2 MSL 默认是 60 秒。如果需要修改，必须修改 Linux 内核代码里 TCP_TIMEWAIT_LEN 的值。

#### 处于 time_wait 状态的连接过多会造成的问题

- 1、每个连接都需要消耗内存。
- 2、每个连接都需要占用端口。一般可以开放的端口为 32768~61000。

如果发起端端口消耗完了，就无法建立新的连接。如果响应端端口消耗完了，理论上可以继续监听新的连接，但是无法处理。

#### 解决处于 time_wait 状态的连接过多的方案

- 1、net.ipv4.tcp_timestamps 和 net.ipv4.tcp_tw_reuse

`net.ipv4.tcp_tw_reuse = 1`，这个只能发起端用。开启后，可以复用处于 time_wait 的连接。应用程序在调用系统函数 connect（[connect(2) - initiate a connection on a socket](https://man7.org/linux/man-pages/man2/connect.2.html)） 时，内核会随机找一个 time_wait 状态超过 1 秒的连接复用。这个选项的前提是 `net.ipv4.tcp_timestamps = 1`，打开 TCP 时间戳支持。

- 2、net.ipv4.tcp_max_tw_buckets

这个值默认为 18000。当处于 time_wait 状态的连接的数量超过这个值时，系统会将后面的新的进入 time_wait 状态的连接的直接关闭。

- 3、通过设置 socket 选项（[setsockopt(2) - get and set options on sockets](https://man7.org/linux/man-pages/man2/setsockopt.2.html)）的 so_linger 参数，来设置调用 close 时的行为。

如果 l_onoff 为非 0， 且 l_linger 值为 0，那么调用 close 后，会立刻该发送⼀个 RST 报文给对端，该 TCP 连接将跳过四次挥手，也就是会跳过 time_wait 状态，直接关闭。

#### 双方同时关闭

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/protocol/tcp/same_time_close.drawio.png">)

[//]: # (</div>)

双方同时关闭的情况有点特殊。两边都认为自己是四次挥手的发起端。在发送 FIN 报文之后，等到了对方的 FIN 报文。

这时双方在应答对方的 FIN 报文后各自进入 closing 状态。这个状态替代了 fin_wait_2 状态。然后双方在接到 ACK 报文后各自进入 time_wait 状态。

### 参考

- [小林coding](https://xiaolincoding.com/)
  - [图解网络](https://xiaolincoding.com/network/)
- [51CTO](https://edu.51cto.com/)
  - [可用行师](https://edu.51cto.com/lecturer/14934648.html)
    - [Linux C核心技术](https://edu.51cto.com/course/28903.html)