---
draft: false
date: 2022-06-01 08:00:00 +0800
lastmod: 2022-06-01 08:00:00 +0800
title: "负载均衡"
summary: "负载均衡"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- architecture-design(架构设计)
---

### 负载均衡

负载均衡（Load Balance）：将负载（工作任务）进行平衡，分摊到多个操作单元上进行运行。

### 轮询

- 轮询算法：将外界的请求轮流的转发给内部的节点。
- 加权轮询算法：将外界的请求根据内部各个节点的权重值，按照一定比例转发。

轮询和加权轮询可以解决普通系统的负载均衡的需求，但是不能解决分布式系统的负载均衡的需求。因为在分布式系统中，每个节点存储的数据是不同的。比如，一个分布式 KV（key-value）缓存系统，某个 key 应该存到哪个（哪些）节点上是确定的，不是访问任意一个节点都可以得到 这个 key 的缓存结果的。

### 哈希

哈希（hash）算法：识别外部请求的特征（key），进行取模运算（Modulo Operation）。理论上相同的特征，每次取模运算都会得到相同的结果。这样就可以将某个特征对应到某个节点，满足负载均衡的需求。

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/architecture-design/load-balance-hash-get.drawio.png">)

[//]: # (</div>)

但是能用哈希进行负载均衡的前提是，参与负载均衡的节点不会扩容或缩容。因为当节点数量发生变化时，取模运算的结果也会跟着变化，这样某个特征就会被对应到到其他的节点上去。

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/architecture-design/load-balance-hash-miss.drawio.png">)

[//]: # (</div>)

想要解决这个问题，就需要进行数据迁移。最坏的情况下，所有的数据都要迁移，数据迁移规模是 O(M)，成本太高。

### 一致性哈希

一致性哈希（consistent hashing）算法可以解决参与负载均衡的节点在扩容或者缩容时，发生过多的数据迁移的问题。

一致哈希算法也用了取模运算，但与哈希算法不同的是，哈希算法是对节点的数量进行取模运算，
而一致哈希算法是对 $2^{32}$ 进行取模运算，其结果是一个固定的值。

一致性哈希要进行两步哈希运算，将数据和节点都映射到一个首尾相连的哈希环上。

- 第一步，对存储节点进行哈希计算，也就是对存储节点做哈希映射，比如根据节点的 IP 地址进行哈希。
- 第二步，当对数据进行存储或访问时，对数据进行哈希映射。对数据进行哈希映射的结果值往顺时针的方向的找到第一个节点，就是存储该数据的节点。

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/architecture-design/load-balance-consistent-hashing-hash-ring.drawio.png">)

[//]: # (</div>)

在一致哈希算法中，如果增加或者移除一个节点，仅影响该节点在哈希环上顺时针相邻的后继节点，其它数据也不会受到影响。但是，一致性哈希算法并不保证节点能够在哈希环上分布均匀。

这两个因素，导致在这种节点分布不均匀的情况下，进行容灾与扩容时，哈希环上的相邻节点容易受到过大影响，容易发生雪崩式的连锁反应。

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/architecture-design/load-balance-consistent-hashing-inhomogeneous.drawio.png">)

[//]: # (</div>)

要想解决节点能在哈希环上分配不均匀的问题，就是要有大量的节点，节点数越多，哈希环上的
节点分布的就越均匀。但问题是，实际中没有那么多节点。

这时可以加入虚拟节点，也就是对一个真实节点做多个副本。具体做法是，不再将真实节点映射到哈希环上，而是将虚拟节点映射到哈希环上，并将虚拟节点映射到实际节点，所以这里有两层映射关系。

[//]: # (<div style="text-align: center; margin: 5px auto">)

[//]: # (<img src="/image/computer-science/architecture-design/load-balance-consistent-hashing-virtual-node.drawio.png">)

[//]: # (</div>)

节点数量多了后，节点在哈希环上的分布就相对均匀了。这时候，如果有访问请求寻址到 A1 这个虚拟节点，接着再通过 A1 虚拟节点找到真实节点 A，这样请求就能访问到真实节点 A 了。在实际的工程中，虚拟节点的数量会大很多，比如 Nginx 的一致性哈希算法，每个权重为 1 的真实节点就含有 160 个虚拟节点。

虚拟节点除了会提高节点的均衡度，还会提高系统的稳定性。当节点变化时，会有不同的节点共同分担系统的变化，因此稳定性更高。比如，当某个节点被移除时，对应该节点的多个虚拟节点均会移除，而这些虚拟节点按顺时针方向的下一个虚拟节点，可能会对应不同的真实节点，即这些不同的真实节点共同分担了节点变化导致的压力。而且，有了虚拟节点后，还可以为硬件配置更好的节点增加权重，比如对权重更高的节点增加更多的虚拟机节点即可。

### 有界负载一致性哈希

当一致性哈希的某个节点过载时，数据会顺位使用下一个节点。

### P2C

P2C 算法：随机选取的两个节点进行打分，选择更优的节点。

- 对新启动的节点使用常量惩罚（penalty），然后使用探针方式的最小化放量进行预热。
- 打分比较低的节点，使用统计衰减的方式，让节点指标逐渐恢复到初始状态（即默认值），避免永久进入黑名单无法恢复。
- 如果发出去的请求超过了预测的延迟时间（predict latency），就会加惩罚。预测的延迟时间可以使用系统指标的滑动平均值配合统计衰减的方式。

### 参考

- 极客时间：Go 进阶训练营
- [The power of two choices in randomized load balancing](https://ieeexplore.ieee.org/document/963420)
