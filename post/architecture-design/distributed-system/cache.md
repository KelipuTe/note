---
draft: true
date: 2022-06-01 08:00:00 +0800
lastmod: 2022-06-01 08:00:00 +0800
title: "分布式缓存"
summary: "分布式缓存"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- architecture-design(架构设计)
- distributed-system(分布式系统)
- cache(缓存)
---

### 缓存->站内链接

### 缓存选型

- Memcache->站内链接
- Redis->站内链接

二者的比较：

- 区别主要在于 Memcache 是多线程模型，Redis 是单（双）线程模型。
- 如果是纯 KV 的场景，用 Memcache 比较好，整体吞吐比较好，不会被大 value 影响。
- Redis 可以发挥非 KV 数据结构的优势，用于数据索引等场景。
- Redis 支持集群（cluster），提供了扩容、缩容、负载均衡的能力。

### 缓存一致性->站内链接

### 多级缓存

如果分布式系统是按细粒度拆分的，那么一个数据聚合的接口可能会大量调用下游系统，也就是说扇出会很大。这时就可以使用多级缓存，在数据聚合的接口的系统里，再加一个缓存，缓存下游系统的数据。

多级缓存特别需要注意的是一致性问题，这里有几个注意点：

- 优先清理下游的数据，因为如果先清理上游的数据，请求打过来，又把下游的脏数据带上来了。
- 下游的缓存的有效时间要大于上游的缓存，防止上游下游的缓存一起失效，直接穿透到数据库。

### 热点缓存

缓存中的某个 key 被非常高频的访问，这就是热点缓存。缓存系统是分布式集群没错，但是针对某个 key，它一定是在某个节点上。所以对热点缓存的请求就会集中命中集群中的某一个节点。导致节点过载甚至崩溃。

可以通过把缓存系统中的远程缓存提升为本地缓存的思路解决热点缓存的问题。本地缓存就是把数据在放在本地内存里，这样后续的请求就可以直接用这个本地内存里的数据，而不用再去请求缓存系统。

- 小表广播：每个服务把自己高频的读多写少的数据，从远程缓存提升为本地缓存。
- 主动监控：服务需要预留人工介入的接口，当某个 key 变成热点时，可以人工把这个 key 变成本地缓存。
- 框架自动识别热点缓存，自动把提升为本地缓存，并持续一小段时间。
- 如果可以提前知道哪个 key 是热点，可以使用多副本的思路，比如 key01、key02、key03。让这些 key 分散到不同的节点上去，相当于分散了以前的那一个节点的压力。

### 参考

- 极客时间：Go 进阶训练营