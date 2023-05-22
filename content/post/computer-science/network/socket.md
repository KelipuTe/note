---
draft: false
create_date: 2021-12-16 08:00:00 +0800
date: 2023-05-15 08:00:00 +0800
title: "网络间进程间通信"
summary: "网络间进程间通信"
toc: true

categories:
- operating-system(操作系统)

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- network(网络)
- linux
- linux-c
- socket
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- VMware Workstation Pro 16
- Ubuntu 22.04
- Linux 5.19.0-32-generic x86_64
- gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0

## 资料

- [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/socket/
- <a href="/drawio/computer-science/network/socket.drawio.html">socket.drawio.html</a>

## 正文

### socket

socket（套接字），就是对 "网络中不同主机上的应用进程之间进行双向通信的端点" 的抽象。

套接字（在一些资料里还称为套接口、数据接口）其实就是一个文件，这个文件是套接字文件描述符。

详细的内容在 Linux 文档 socket(7) 里面。

> socket(7)
> </br></br>
> socket(2) creates a socket,
> connect(2) connects a socket to a remote socket address,
> the bind(2) function binds a socket to a local socket address,
> listen(2) tells the socket that new connections shall be accepted,
> and accept(2) is used to get a new socket with a new incoming connection.
> </br></br>
> send(2), sendto(2), and sendmsg(2) send data over a socket,
> and recv(2), recvfrom(2), recvmsg(2) receive data from a socket.
> poll(2) and select(2) wait for arriving data or a readiness to send data.
> In addition, the standard I/O operations like write(2), writev(2), sendfile(2),
> read(2), and readv(2) can be used to read and write data.
> </br></br>
> getsockopt(2) and setsockopt(2) are used to set or get socket layer or protocol options.
> </br></br>
> close(2) is used to close a socket.

基于 Linux 一切皆文件的理念，在内核中 socket 也是以文件的形式存在的。进程打开的 socket 是可以在进程的 fd 目录下看到的。

每个进程都有一个 task_struct 数据结构，该结构体里有一个指向文件描述符数组的成员指针。该数组里列出这个进程打开的所有文件的文件描述符。

数组的下标是文件描述符，是一个整数，而数组的内容是一个指针，指向内核中所有打开的文件的列表，也就是说内核可以通过文件描述符找到对应打开的文件。

每个文件都有一个 inode。socket 文件的 inode 指向了内核中的 socket 结构，在这个结构体里有两个队列，分别是发送队列和接收队列。

这个两个队列里面保存的是一个个 struct sk_buff，这些 sk_buff 用双向链表的组织形式串起来。

### sk_buff

sk_buff 可以表示各个层的数据包，在应用层数据包叫 data，在 TCP 层我们称为 segment，在 IP 层叫 packet，在数据链路层称为 frame。

这样设计的原因是：协议栈采用的是分层结构，上层向下层传递数据时需要增加包头，下层向上层数据时又需要去掉包头。

如果每一层都用一个结构体，那在层之间传递数据的时候，就要发生多次拷贝，这将大大降低 CPU 效率。

调整 sk_buff 中 data 的指针，可以实现在层级之间传递数据时，不发生拷贝，只用 sk_buff 一个结构体来描述所有的网络包。

比如：当接收报文时，从网卡驱动开始，通过协议栈层层往上传送数据报，通过增加 skb->data 的值，来逐步剥离协议首部。

当要发送报文时，创建 sk_buff 结构体，数据缓存区的头部预留足够的空间，用来填充各层首部，在经过各下层协议时，通过减少 skb->data 的值来增加协议首部。

见图：**socket.drawio.html 4-2**。

### socket 编程的大概流程

见图：**socket.drawio.html 2-2**。

### 创建 socket

> socket(2)</br>
> </br>
> AF_INET      IPv4 Internet protocols                    ip(7)</br>
> AF_INET6     IPv6 Internet protocols                    ipv6(7)</br>
> </br>
> SOCK_STREAM</br>
> Provides sequenced, reliable, two-way, connection-based byte streams.
> An out-of-band data transmission mechanism may be supported.</br>
> </br>
> SOCK_DGRAM</br>
> Supports datagrams (connectionless, unreliable messages of a fixed maximum length).</br>
> </br>
> SOCK_NONBLOCK</br>
> Set the O_NONBLOCK file status flag on the open file description
> (see open(2)) referred to by the new file descriptor.
> Using this flag saves extra calls to fcntl(2) to achieve the same result.

创建 socket 的时候，可以选择创建 ipv4 还是 ipv6 的（还有 UNIX 等），TCP 还是 UDP 的（还有 RAW 等）。

SOCK_STREAM 就是 TCP，提供有顺序、可靠、双向、基于连接的字节流。

SOCK_DGRAM 就是 UDP，支持数据包（最大长度固定的无连接、不可靠的消息）

OCK_NONBLOCK 表示非阻塞模式，调用 recv() 等的时候，不会阻塞。

下面的代码示例用的是，网络协议为 IPv4，传输协议为 TCP 的 socket。

### TCP

TCP（Transmission Control Protocol，传输控制协议），可靠。

详细的内容在 Linux 文档 tcp(7) 里面。

#### bind()

socket 创建好之后，调用 bind 函数，给 socket 绑定一个 IP 地址和端口。

绑定 IP 地址的目的，一台机器是可以有多个网卡的，每个网卡都有对应的 IP 地址，当绑定一个网卡时，内核在收到该网卡上的包，才会发给应用程序。

绑定端口的目的，当内核收到 TCP 报文，通过 TCP 头里面的端口号，来找到应用程序，然后把数据传递过来。

注意 bind() 系统调用的参数 "const struct sockaddr *addr"。这玩意在 socket 的 family 参数不一样的时候，是有区别的。

> bind(2)</br>
> The rules used in name binding vary between address families.
> Consult the manual entries in Section 7 for detailed information.
> For AF_INET, see ip(7); for AF_INET6, see ipv6(7); for AF_UNIX, see unix(7);

ipv4 用的数据结构长这样。

```c
struct sockaddr_in {
    sa_family_t    sin_family; /* address family: AF_INET */
    in_port_t      sin_port;   /* port in network byte order */
    struct in_addr sin_addr;   /* internet address */
};
```

用的时候这样用。

```c
struct sockaddr_in serverAddr;
serverAddr.sin_family = AF_INET;
serverAddr.sin_port = 0x1d25;
serverAddr.sin_addr.s_addr = INADDR_ANY;

bind(sockfd, (struct sockaddr *)&serverAddr, sizeof(serverAddr));
```

#### 主机字节序和点分十进制

上面在设置 socket 的时候，地址是 INADDR_ANY，端口是 0x1d25。这里用的是系统提供的常量和十六进制。

sin_port 参数，它是网络字节序的，这里的 0x1d25 设置的是 9501 端口。INADDR_ANY 就是点分十进制的 0.0.0.0。

9501（十进制）、00100101 00011101（二进制）、0x251d（十六进制）、0x1d25（网络字节序）

如果想直观的直接用 0.0.0.0 和 9501 进行设置。需要用到 htons() 和 inet_addr()。

htons() 把 16 位的主机字节序转换为网络字节序的。inet_addr() 把点分十进制的 ipv4 地址转换为二进制网络字节顺序的。

```c
serverAddr.sin_port = htons(9501);
serverAddr.sin_addr.s_addr = inet_addr("0.0.0.0");
```

#### 端口占用

再重复测试的时候，可能会遇到端口占用的报错，"errno=98, error=Address already in use"。

这个时候可以通过 setsockopt()，设置 socket 选项为 SO_REUSEPORT，来复用端口。另外，SO_REUSEADDR，可以用于重用地址。

> socket(7)</br>
> The socket options listed below can be set by using setsockopt(2)
> and read with getsockopt(2) with the socket level set to SOL_SOCKET for all sockets.</br>
> </br>
> SO_REUSEPORT</br>
> Permits multiple AF_INET or AF_INET6 sockets to be bound to an identical socket address.
> This option must be set on each socket (including the first socket) prior to calling bind(2) on the socket.</br>
> </br>
> SO_REUSEADDR</br>
> Indicates that the rules used in validating addresses supplied in a bind(2) call should allow reuse of local addresses.

在调用 bind() 之前先调用 setsockopt() 对 socket 进行设置。

```c
int sockfd = socket(AF_INET, SOCK_STREAM, 0);
setsockopt(sockfd, SOL_SOCKET, SO_REUSEPORT, &report, sizeof(report));
bind(sockfd, (struct sockaddr *)&serverAddr, sizeof(serverAddr));
```

#### listen()

绑定完 IP 地址和端口后，就可以调用 listen() 进行监听。如果要判定服务器中一个网络程序有没有启动，可以通过 netstat 命令查看对应的端口号是否有被监听。

> listen(2)</br>
> The backlog argument defines the maximum length to which the queue of pending connections for sockfd may grow.

listen() 的 backlog 参数定义 sockfd 的挂起连接队列的最大长度。现在一般认为 backlog 参数是 Accept 队列的大小。

早期的 backlog 参数是指 SYN 队列的大小。Accept 队列的长度 = min(backlog, somaxconn)。somaxconn 是一个内核参数。

#### accept()

服务端进入了监听状态后，通过调用 accept()，来从内核获取客户端的连接，如果没有客户端连接，则会阻塞等待客户端连接的到来。

客户端在创建好 socket 后，调用 connect() 发起连接，该函数的参数要指明服务端的 IP 地址和端口号，然后 TCP 三次握手就开始了。

客户端调用 connect() 成功返回，说明第二次握手完成。服务端调用 accept() 成功返回，说明第三次握手完成。

在 TCP 连接的过程中，服务器的内核实际上为每个 Socket 维护了两个队列：

- 一个是还没完全建立连接的队列，称为 TCP 半连接队列，这个队列都是没有完成三次握手的连接，此时服务端处于 syn_rcvd 的状态；
- 一个是已经建立连接的队列，称为 TCP 全连接队列，这个队列都是完成了三次握手的连接，此时服务端处于 established 状态；

当 TCP 全连接队列不为空后，服务端调用 accept() 时，就会从内核中的 TCP 全连接队列里拿出一个已经完成连接的 socket 返回应用程序，后续数据传输都用这个 socket。

监听的 socket 和真正用来传数据的 socket 是两个：一个叫作监听 socket；一个叫作已连接 socket。

#### read()、write()

连接建立后，客户端和服务端就开始相互传输数据了，双方都可以通过调用 read() 和 write() 来读写数据。

#### close()

服务端接收到 FIN 报文后，会在接收缓冲区中插入一个 EOF。read() 可以从缓冲区中读到 EOF 标志。

代码示例：

- 一次性的 TCP 服务端：**{demo-c}/demo-in-linux/socket/tcp/server_once.c**
- 一次性的 TCP 客户端：**{demo-c}/demo-in-linux/socket/tcp/client_once.c**

### 进程的 TCP socket 网络表

启动 server_multiple.c 服务端，得到进程号。然后，查看进程的内存目录里的 "/proc/{pid}/fd/"。

```
total 0
lrwx------ 1 root root 64 Dec 18 05:53 0 -> /dev/pts/1
lrwx------ 1 root root 64 Dec 18 05:53 1 -> /dev/pts/1
lrwx------ 1 root root 64 Dec 18 05:53 2 -> /dev/pts/1
lrwx------ 1 root root 64 Dec 18 05:53 3 -> socket:[20443]
```

这个 "3 -> socket:\[20443\]"，就是 socket() 创建的。

然后，再查看 "/proc/{pid}/net/tcp" 文件。这个就是 TCP 的 socket 网络表。

```
sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode

0: 00000000:251D 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 20443 1 0000000000000000 100 0 0 10 0
```

- local_address，前面是本地 ip，网络字节序，后面是本地端口，主机字节序。
- rem_address，前面是远端 ip，网络字节序，后面是远端端口，主机字节序。
- st（connection state），套接字状态，16 进制，具体解释看下面。

st 字段的状态（10 进制）：

- TCP_ESTABLISHED:1   
- TCP_SYN_SENT:2
- TCP_SYN_RECV:3      
- TCP_FIN_WAIT1:4
- TCP_FIN_WAIT2:5     
- TCP_TIME_WAIT:6
- TCP_CLOSE:7         
- TCP_CLOSE_WAIT:8
- TCP_LAST_ACL:9     
- TCP_LISTEN:10
- TCP_CLOSING:11

使用 telnet 命令创建一个连接，`telnet 127.0.0.1 9501`。

然后，再去查看 "/proc/{pid}/fd/" 目录。

```
total 0
lrwx------ 1 root root 64 Dec 18 05:53 0 -> /dev/pts/1
lrwx------ 1 root root 64 Dec 18 05:53 1 -> /dev/pts/1
lrwx------ 1 root root 64 Dec 18 05:53 2 -> /dev/pts/1
lrwx------ 1 root root 64 Dec 18 05:53 3 -> socket:[20443]
lrwx------ 1 root root 64 Dec 18 05:53 4 -> socket:[20444]
```

和之前比，这里多了一个 "4 -> socket:\[20444\]"，这就是 accept() 创建的，对应连接上来的 TCP。

再看一眼 "/proc/{pid}/net/tcp" 文件。

```
sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode

0: 00000000:251D 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 20443 1 0000000000000000 100 0 0 10 0
2: 0100007F:251D 0100007F:DA50 01 00000000:00000000 00:00000000 00000000     0        0 20444 1 0000000000000000 20 0 0 10 -1
```

和之前比，这里多了 telnet 命令连接上来之后的一些数据，通过 local_address 和 rem_address 字段可以判断服务端和客户端。

用 uid 也可以对上，上面的 "socket:\[20443\]" 和 "socket:\[20444\]" 就是这里的 20443 和 20444。

### HTTP

HTTP（Hyper Text Transfer Protocol，超文本传输协议），是一个简单的 "请求-响应" 协议，它通常运行在 TCP 之上。

在 TCP 的基础之上，对连接上来的客户端响应符合 HTTP 协议格式的数据，就可以实现简单的 HTTP 响应。

代码示例：

- 一次性的 HTTP 服务端：**{demo-c}/demo-in-linux/socket/http/server_once.c**
- 可重用的 HTTP 服务端：**{demo-c}/demo-in-linux/socket/http/server_multiple.c**

### UDP

UDP（User Datagram Protocol，用户数据包协议），不可靠，而且数据报会被重新排序。

详细的内容在 Linux 文档 udp(7) 里面。

注意，UDP 服务端不需要调用 listen()、connect()、accept()，因为 UDP 是不建立连接的。

代码示例：

- 一次性的 UDP 服务端：**{demo-c}/demo-in-linux/socket/udp/server_once.c**
- 一次性的 UDP 客户端：**{demo-c}/demo-in-linux/socket/udp/client_once.c**
- 可重用的 UDP 服务端：**{demo-c}/demo-in-linux/socket/udp/server_multiple.c**

### 进程的 UDP socket 网络表

基本和 TCP 那里差不多，区别是 UDP 的文件是 "/proc/{pid}/net/udp"。

### TCP 和 UDP 的区别

TCP 调用 recv() 时，如果对端发送了多次，缓冲区有多少数据就读多少，不会丢失数据。

UDP 调用 recvfrom() 时，如果对端发送了多次，后面的数据会被丢弃。当发送端调用 sendto() 的次数和接收端调用 recvfrom() 的次数一样时才有可能获取完整的数据。

### UNIX socket

UNIX socket 只能用于同一台机器上的进程间通信。ipv4 的 TCP 和 UDP 需要走网卡，UNIX socket 不需要。

UNIX socket 的类型也有 TCP 和 UDP 两种，但是 UNIX socket 的 UDP 是可靠的，而且数据报不会重新排序。

详细的内容在 Linux 文档 unix(7) 里面。

注意，UNIX socket 还分匿名的和命名的，这两个创建方式不一样。

匿名的，要用 socketpair() 创建一对 socket。命名的，和 ipv4 的 TCP、UDP 使用起来差不多。

代码示例：

- 匿名 UNIX socket：**{demo-c}/demo-in-linux/unix-socket/unnamed/unnamed.c**


> socket(2)</br>
> AF_LOCAL     Synonym for AF_UNIX

命名的，创建 socket 的时候，socket() 的 domain 参数和上面 ipv4 的也不一样了，UNIX socket 是 AF_LOCAL。

绑定的时候也不是绑端口了，而是绑定文件描述符。而且如果想交互的话，服务端和客户端，需要各自创建自己的文件描述符。

注意，一定是，服务端和客户端各一个。在 ipv4 的 TCP 和 UDP 里面，这两个文件描述符，分别是 socket() 和 accept() 创建的。

代码示例：

- 一次性的 TCP 服务端：**{demo-c}/demo-in-linux/unix-socket/tcp/server_once.c**
- 一次性的 TCP 客户端：**{demo-c}/demo-in-linux/unix-socket/tcp/client_once.c**
- 一次性的 UDP 服务端：**{demo-c}/demo-in-linux/unix-socket/udp/server_once.c**
- 一次性的 UDP 客户端：**{demo-c}/demo-in-linux/unix-socket/udp/client_once.c**

## 参考（reference）

- {小林coding}/[图解系统](https://xiaolincoding.com/os/)
  - 9.2 I/O 多路复用：select/poll/epoll
- [linux 网络 sk_buff结构](https://blog.csdn.net/wwwlyj123321/article/details/127715903)
- {51CTO学堂}/{可用行师}/[Linux C核心技术](https://edu.51cto.com/course/28903.html)
  - 网络间进程间通信部分、socket 部分、TCP 部分、UDP 部分
- [linux /proc/net/tcp 文件分析](https://blog.csdn.net/whatday/article/details/100693051)
- [ChatGPT](https://chat.openai.com/) + [DeepL](https://www.deepl.com/translator)
