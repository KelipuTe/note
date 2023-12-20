---
draft: true
create_date: 2022-01-14 08:00:00 +0800
date: 2023-05-21 08:00:00 +0800
title: "I/O 模型"
summary: "五种 I/O 模型；I/O 多路复用；epoll；"
toc: true

categories:
- network(网络)

tags:
- computer-science(计算机科学)
- network(网络)
- operating-system(操作系统)
- linux
- linux-c
- socket
- epoll
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- VMware Workstation Pro 16
- Ubuntu 22.04
- Linux 5.19.0-32-generic x86_64
- gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0

前置笔记：[网络间进程间通信](/计算机/network/socket)

## 资料

- [{demo-c}](https://github.com/KelipuTe/demo-c)/demo-in-linux/epoll/
- <a href="/drawio/computer-science/network/io_model.drawio.html">io_model.drawio.html</a>

## 正文

### 同步、异步

用户线程与内核的交互方式。

- 同步：用户线程发起 I/O 请求后，需要等待或者轮询，内核 I/O 操作完成后才能继续执行。
- 异步：用户线程发起 I/O 请求后继续执行，当内核操作完成后会通知用户线程，或者调用用户线程注册的回调函数。

### 阻塞、非阻塞

- 阻塞：执行某个操作，导致一直处于等待状态。
- 非阻塞：执行某个操作后立即返回一个状态。

阻塞 I/O 等待的是两个过程：内核准备好数据、数据从内核态拷贝到用户态。见图 **io_model.drawio.html 2-2**。

### 同步阻塞 I/O

用户线程通过系统调用 read 发起 I/O 操作，由用户空间转到内核空间。内核等到数据包到达以后，将接收的数据拷贝到用户空间，完成 read。

用户需要等待 read 将 socket 中的数据读取到缓冲区后，才继续处理接收的数据。整个 I/O 请求过程中，用户线程是被阻塞的，用户发起请求时，不能做任何事情。

在 Linux 中，创建的 socket 默认都是同步阻塞的。

见图 **io_model.drawio.html 4-2**。

基于最原始的阻塞网络 I/O，如果服务器要支持多个客户端，其中比较传统的方式，就是使用多进程模型。也就是为每个客户端分配一个进程来处理请求。

见图 **io_model.drawio.html 4-4**。

服务器的主进程负责监听客户的连接，一旦与客户端连接完成，accept 函数就会返回一个已连接 socket。

这时通过 fork() 创建一个子进程，把父进程所有相关的东西都复制一份，包括文件描述符、内存地址空间、程序计数器、执行的代码等。

这两个进程刚复制完的时候，几乎一摸一样。于是子进程就可以直接使用已连接 socket 和客户端通信。

父进程不需要关心已连接 socket，只需要关心监听 socket。子进程不需要关心监听 socket，只需要关心已连接 socket。

用多个进程来应付多个客户端的方式，在应对 100 个客户端还是可行的，但是每产生一个进程，必会占据一定的系统资源。

而且进程间上下文切换不仅包含了虚拟内存、栈、全局变量等用户空间的资源，还包括了内核堆栈、寄存器等内核空间的资源。数量一多，性能会大打折扣。

既然进程切换代价过重，那么就可以使用多线程模型。

见图 **io_model.drawio.html 4-6**。

线程是运行在进程中的一个逻辑流，单进程中可以运行多个线程，同进程里的线程可以共享进程的部分资源的。比如，文件描述符列表、进程空间、代码、全局数据、堆、共享库等。

这些共享些资源在上下文切换时是不需要切换，而只需要切换线程的私有数据、寄存器等不共享的数据，因此同一个进程下的线程上下文切换的开销要比进程小得多。

当服务器与客户端 TCP 完成连接后，通过 pthread_create() 创建线程，然后将已连接 socket 的文件描述符传递给线程函数。

接着在线程里和客户端进行通信，从而达到并发处理的目的。需要注意的是，这个队列是全局的，每个线程都会操作，为了避免多线程竞争，需要加入锁机制。

如果每来一个连接就创建一个线程，线程运行完后，还得操作系统还得销毁线程，虽说线程切换的上下文开销不大。

但是如果频繁创建和销毁线程，系统开销也是不小的。这时就可以使用线程池的方式来避免线程的频繁创建和销毁。

### 同步非阻塞 I/O

用户线程可以在发起 I/O 请求后立即返回，但并未读取到任何数据。用户线程需要不断发起 I/O 请求，直到数据到达后，才真正读取到数据。

在整个 I/O 请求的过程中，虽然用户线程每次发起 I/O 请求后可以立即返回。但是为了等到数据，仍需要不断地轮询、重复请求。会消耗大量 CPU 资源。

见图 **io_model.drawio.html 6-2**。

在 Linux 中，创建 socket 的时候设置为 `NONBLOCK` 就是同步非阻塞。

### 多路复用

多路复用也叫时分多路复用。它允许一个进程同时管理 I/O 操作，而不需要多个线程或进程。

多路复用技术的目的是，使用一个进程来维护多个 socket。解决多进程模型和多线程模型，连接数过多时的系统负担问题。

一个进程虽然任一时刻只能处理一个请求，但是如果处理每个请求的耗时控制在 1 毫秒以内，这样 1 秒内就可以处理上千个请求。把时间拉长，看上去就是多个请求复用了一个进程。

Linux 内核态提供 select、poll、epoll 三种多路复用系统调用给用户态。

#### select

多路分离函数 select 可以避免同步非阻塞 I/O 模型中轮询等待的问题。

用户将需要进行 I/O 操作的 socket 添加到 select 中，然后阻塞等待 select 系统调用返回。

当数据到达时，socket 被激活后返回，用户线程调用 read 读取数据并继续执行。

这里需要注意的是，select 只会通知用户线程有数据来了，但是不知道是哪一个 socket。

select 将已连接的 socket 都放到一个文件描述符集合，然后调用 select 函数将文件描述符集合拷贝到内核里，让内核来检查是否有网络事件产生。

检查的方式很粗暴，就是通过遍历文件描述符集合的方式。当检查到有事件产生后，将此 socket 标记为可读或可写。

接着再把整个文件描述符集合拷贝回用户态里，然后用户态还需要再通过遍历的方法找到可读或可写的 socket，然后再对其处理。

见图 **io_model.drawio.html 8-2**。

对于 select 这种方式，需要遍历 2 次文件描述符集合，一次是在内核态里，一个次是在用户态里。

而且，还会发生 2 次拷贝文件描述符集合，先从用户空间传入内核空间，由内核修改后，再传出到用户空间中。

使用 select 进行 I/O 请求与同步阻塞模型并无太大区别。同步阻塞模型中，必须使用多线程。

select 的优势主要在于用户可以在一个线程内同时处理多个 socket 的 I/O 请求。用户可以注册多个 socket，然后不断调用 select 读取被激活的 socket。

#### poll

poll 和 select 并没有太大的本质区别，都是使用线性结构存储进程关注的 socket 集合，因此都需要遍历文件描述符集合来找到可读或可写的 socket。

时间复杂度为 O(n)，而且也需要在用户态与内核态之间拷贝文件描述符集合，这种方式随着并发数上来，性能的损耗会呈指数级增长。

#### epoll

epoll 没有对描述符数目的限制，它所支持的文件描述符上限是整个系统最大可以打开的文件数目。epoll 只有 epoll_create()、epoll_ctl()、epoll_wait() 这三个系统调用。

epoll_create 可以创建 epoll 句柄，epoll_ctl 可以添加、修改、删除要监控的文件描述符。

调用 epoll_wait 和调用 select 差不多，调用之后就阻塞并等待返回。和 select 不同的是，epoll_wait 返回的时候，只会携带有数据的文件描述符。

epoll 通过两个方面，解决了 select 和 poll 的问题。

- 第一点，epoll 在内核里使用红黑树来跟踪进程所有待检测的文件描述符，把需要监控的 socket 通过 epoll_ctl 函数加入内核中的红黑树里。
  这样就不需要像 select 每次操作时都传入整个socket 集合，只需要传入一个待检测的 socket，减少了内核和用户空间大量的数据拷贝和内存分配。
- 第二点，epoll 使用事件驱动的机制，内核里维护了一个链表来记录就绪事件，当某个 socket 有事件发生时，通过回调函数内核会将其加入到这个就绪事件列表中。
  当用户调用 epoll_wait 函数时，只会返回有事件发生的文件描述符的个数，不需要像 select 那样轮询扫描整个 socket 集合。

见图 **io_model.drawio.html 8-4**。

epoll 的方式即使监听的 socket 数量越多的时候，效率也不会大幅度降低，能够同时监听的 socket 的数目也非常的多，上限就为系统定义的进程打开的最大文件描述符个数。

#### epoll_create()

epoll_create() 用于创建一个 epoll 文件描述符。创建的 epoll 文件描述符可以在 "/proc/{pid}/fd/" 目录下看到。

> epoll_create(2)</br>
> epoll_create() returns a file descriptor referring to the new epoll instance.</br>
> This file descriptor is used for all the subsequent calls to the epoll interface.</br>
> When no longer required, the file descriptor returned by epoll_create() should be closed by using close(2).</br>
> When all file descriptors referring to an epoll instance have been closed,</br>
> the kernel destroys the instance and releases the associated resources for reuse.

epoll 底层是红黑树实现的，epoll 文件描述符会关联 Linux 内核事件表。

在使用完 epoll 文件描述符后，必须调用 close() 关闭，否则可能导致 fd 被耗尽。

#### epoll_ctl()

> epoll_ctl(2)</br>
> </br>
> EPOLL_CTL_ADD</br>
> Add an entry to the interest list of the epoll file descriptor, epfd.</br>
> EPOLL_CTL_MOD</br>
> Change the settings associated with fd in the interest list to the new settings specified in event.</br>
> EPOLL_CTL_DEL</br>
> Remove (deregister) the target file descriptor fd from the interest list.

epoll_ctl() 可用于添加、修改、删除 epoll 文件描述符关联的 Linux 内核事件。

在 socket 编程这里，一般是把监听的 socket 的那个文件描述符添加到 epoll 中，并关注 EPOLLIN 事件。

> epoll_ctl(2)</br>
> </br>
> EPOLLIN</br>
> The associated file is available for read(2) operations.</br>
> EPOLLOUT</br>
> The associated file is available for write(2) operations.

epoll_ctl() 的 event 参数是 epoll_event 结构体。把 epoll_event 结构体的 events 参数设置为 EPOLLIN，就表示要关注的是 EPOLLIN 事件。

代码一般这样写。这里就是让 epoll 去监听 socket 的文件描述符上的 EPOLLIN 事件。

```
struct epoll_event event;
event.events = EPOLLIN;
event.data.fd = sockfd;
epoll_ctl(epollFD, EPOLL_CTL_ADD, sockfd, &event);
```

一般 EPOLLIN 事件发生的时候，通常是：客户端连接了、客户端断开了、出现异常了、客户端发送的数据达到内核的相关设置（接收水位标志），简单理解就是，对应的 fd 可以 read() 了。

EPOLLOUT 一般是用在，TCP 连接发送数据发送的不完整的时候。比如，缓冲区中的数据只发出去一半。

我目前就正常进用过。这个时候会把 events 参数设置设置为 "EPOLLIN | EPOLLOUT"。下一轮紧接着就继续发数据。

#### epoll_wait()

epoll_wait() 可以等待 epfd 参数对应的 epoll 文件描述符上的事件。

如果 epoll_wait() 执行成功的话，返回值就是得到的事件的数量。如果有事件，则会以数组的形式放到 events 参数里。

timeout 参数可以控制等待的方式：-1=阻塞调用；0=非阻塞；>0=等待几秒返回。

代码一般这样写。如果执行成功，那么 epollWaitResult 就是事件的数量。事件的详细数据会被放到 a5epollEvent 里面。

```
struct epoll_event a5epollEvent[1024];
int epollWaitResult = epoll_wait(epollFD, a5epollEvent, 1024, -1);
```

epoll_event 长这样，我们需要的是 data 里面的 fd。这个 fd 就是触发了 EPOLLIN 事件的文件描述符。

```
struct epoll_event {
    uint32_t     events;    /* Epoll events */
    epoll_data_t data;      /* User data variable */
};
typedef union epoll_data {
    void    *ptr;
    int      fd;
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;
```

在 socket 编程这里，如果事件对应的文件描述符，就是 socket 的文件描述符，那就说明这个事件是客户端连接。

accept() 拿到 TCP 连接的 connfd 文件描述符之后，把它也添加到 epoll 中，并关注 EPOLLIN 事件。

然后，epoll_wait() 就会等到 connfd 文件描述符的 EPOLLIN 事件。这就对应着客户端给服务端发数据过来了。

如果 read() 不到东西，那大概是对方已经关闭了连接，那就可以就把 TCP 连接的 connfd 文件描述符从 epoll 中删除了。

代码示例：**{demo-c}/demo-in-linux/epoll/epoll.c**

### reactor 模式

reactor 模式：基于面向对象的思想，对 I/O 多路复用作一层封装，让使用者不用考虑底层网络 API 的细节，只需要关注应用代码的编写。

reactor（翻译：核反应堆），这里应该是指对事件的反应。reactor 模式也叫 dispatcher 模式。

通过多路复用监听事件，然后在收到事件后，根据事件类型 Dispatch（分配）给某个进程（线程）。

reactor 模式主要由两个核心部分组成：reactor 、资源池（如，线程池）。reactor 负责监听和分发事件，事件类型包含连接事件、读写事件。资源池负责处理事件。

reactor 模式是灵活多变的，reactor 的数量可以只有一个，也可以有多个，资源池可以只有单个进程（线程），也可以是多个进程（线程）。可以灵活应对不同的业务场景。

reactor 可以理解为，来了事件，操作系统通知应用进程，让应用进程来处理。

#### 单 reactor 单进程（线程）

- reactor 对象，监听和分发事件；
- acceptor 对象，获取连接；
- handler 对象，处理业务；

reactor 对象通过多路复用接口监听事件，收到事件后根据事件的类型通过 dispatch 进行分发给 acceptor 对象或者 handler 对象。

如果是连接建立的事件，则交由 acceptor 对象进行处理，acceptor 对象会通过 accept 获取连接。

然后，acceptor 对象会构建一个 handler 对象来处理后续业务。如果不是连接建立事件，则交由当前连接对应的 handler 对象来处理事件。

见图 **io_model.drawio.html 10-2**。

单 reactor 单进程的方案因为全部工作都在同一个进程内完成，所以实现起来比较简单，不需要考虑进程间通信，也不用担心多进程竞争。但是，这种方案存在 2 个缺点：

- 因为只有一个进程，无法充分利用 多核 CPU 的性能；
- handler 对象在业务处理时，整个进程是无法处理其他连接的事件的，如果业务处理耗时比较长，那么就造成响应的延迟；

单 reactor 单进程的方案不适用计算机密集型的场景，只适用于业务处理非常快速的场景。

采用单 reactor 单进程的方案的有 Redis。

#### 单 reactor 多进程（线程）

- reactor 对象，监听和分发事件；
- acceptor 对象，获取连接；
- handler 对象，处理业务；

reactor 对象通过多路复用接口监听事件，收到事件后根据事件的类型通过 dispatch 进行分发给 acceptor 对象或者 handler 对象。

如果是连接建立的事件，则交由 acceptor 对象进行处理，acceptor 对象会通过 accept 获取连接。然后，acceptor 对象会构建一个 handler 对象来处理后续业务。

前三个步骤和单 reactor 单进程（线程）一样，不一样的是 handler 对象。handler 对象不再负责业务处理，只负责数据的接收和发送。

handler 对象通过 read 读取到数据后，会将数据发给子线程里的 processor 对象进行业务处理。

子线程里的 processor 对象就进行业务处理，处理完后，将结果发给主线程中的 handler 对象，接着由 handler 通过 write 方法将响应结果发送给 client。

见图 **io_model.drawio.html 10-4**。

单 reator 多线程的方案优势在于能够充分利用多核 CPU。

那既然引入多线程，自然就带来了多线程竞争资源的问题。例如，子线程完成业务处理后，要把结果传递给主线程的 handler 进行发送，这里涉及共享数据的竞争。

要避免多线程由于竞争共享资源而导致数据错乱的问题，就需要在操作共享资源前加上互斥锁。

保证任意时间里只有一个线程在操作共享资源，待该线程操作完释放互斥锁后，其他线程才有机会操作共享数据。

一个 reactor 对象承担所有事件的监听和响应，而且只在主线程中运行，在面对瞬间高并发的场景时，容易成为性能的瓶颈的地方。

#### 多 reactor 多进程（线程）

主线程中的主 reactor 对象通过多路复用接口监听连接建立事件，收到事件后通过 acceptor 对象中的 accept 获取连接，将新的连接分配给某个子线程。

子线程中的子 reactor 对象将主 reactor 对象分配的连接加入多路复用接口继续进行监听，并创建一个 handler 用于处理连接的响应事件。

如果有新的事件发生时，子 reactor 会调用当前连接对应的 handler 对象来进行响应。

见图 **io_model.drawio.html 10-6**。

多 reactor 多线程的方案看起来复杂，实际实现时，比单 reactor 多线程的方案要简单。

- 主线程和子线程分工明确，主线程只负责接收新连接，子线程负责完成后续的业务处理。
- 主线程和子线程的交互很简单，主线程只需要把新连接传给子线程，子线程无须返回数据，直接就可以在子线程将处理结果发送给客户端。

采用多 reactor 多线程的方案的有 Memcache，Nginx（不完全是）。

Nginx 的方案在主进程中仅仅用来初始化 socket，并没有创建主 reactor 来 accept 连接，而是由子进程的 reactor 来 accept 连接。

通过锁来控制一次只有一个子进程进行 accept（防止出现惊群现象），子进程 accept 新连接后就放到自己的 reactor 进行处理，不会再分配给其他子进程。

### 信号驱动 I/O

当用户线程发起一个 I/O 请求操作，会给对应的 socket 注册一个信号函数，然后用户线程会继续执行。、

当内核数据就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在信号函数中调用 I/O 读写操作来进行实际的 I/O 请求操作。

### 异步 IO

当用户线程发起 read 操作之后，立刻就可以开始去做其它的事。

另一方面，从内核的角度，当它收到一个 read 之后，它会立刻返回，说明 read 请求已经成功发起了，因此不会对用户线程产生任何阻塞。

内核会等待数据准备完成，然后将数据拷贝到用户线程。当这一切都完成之后，内核会给用户线程发送一个信号，告诉它 read 操作完成了。

也就说用户线程完全不需要关心实际的整个 I/O 操作是如何进行的，只需要先发起一个请求，当接收内核返回的成功信号时表示 I/O 操作已经完成，可以直接去使用数据了。

在 Linux 下的异步 I/O 是不完善的， AIO 系列函数是由 POSIX 定义的异步操作接口，不是真正的操作系统级别支持的，而是在用户空间模拟出来的异步。

并且仅支持基于本地文件的 AIO 异步操作，网络编程中的 socket 是不支持的，这也使得基于 Linux 的高性能网络程序都是使用 reactor 方案。

### proactor 模式

- 1，proactor initiator 负责创建 proactor 和 handler 对象，并将 proactor 和 handler 都通过 asynchronous operation processor 注册到内核；
- 2，asynchronous operation processor 负责处理注册请求，并处理 IO 操作；
- 3，asynchronous operation processor 完成 I/O 操作后通知 proactor；
- 4，handler 完成业务处理；

见图 **io_model.drawio.html 12-2**。

proactor 可以理解为来了事件操作系统来处理，处理完再通知应用进程。

## 参考（reference）

- {小林coding}/[图解系统](https://xiaolincoding.com/os/)
  - 9.2 I/O 多路复用：select/poll/epoll
  - 9.3 高性能网络模式：Reactor 和 Proactor
- [100%弄明白5种IO模型](https://zhuanlan.zhihu.com/p/115912936)
- [ChatGPT](https://chat.openai.com/) + [DeepL](https://www.deepl.com/translator)
