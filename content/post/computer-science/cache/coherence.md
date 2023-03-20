---
draft: false
date: 2022-06-02 08:00:00 +0800
lastmod: 2022-06-02 08:00:00 +0800
title: "缓存一致性"
summary: "缓存一致性"
toc: true

categories:
- data-structure(数据结构)

tags:
- computer-science(计算机科学)
- cache(缓存)
---

### 缓存刷新策略

- 更新数据库后删除缓存

删除缓存的问题在于，如果删除的缓存是热点缓存，那么大量的请求会透传到数据库，有一定的风险。

一个解决方案是，缓存系统把这个删掉的缓存暂存一小段时间。请求过来时，返回这个过期的数据，并标记出来是过期的数据，并且触发回填操作。前端在收到这个过期的数据后，可以自行判断要不要使用。

- 更新数据库后更新缓存

更新缓存的问题在于，如果并发的两个请求都修改了缓存的值。在数据库层面，可以通过加锁，保证顺序是正确的，但是回填缓存的时候，由于网络等原因，这个顺序就无法保证了。

### 最终一致性

由请求产生的更新缓存的操作是有可能会失败的，比如因为网络原因连不上等，所以有可能存在缓存不一致的情况。这个时候就需要一个兜底的补偿机制，保证最终一致性。

可以开一个服务，订阅数据库的日志，拿到数据后去缓存系统回放。这样如果由请求触发的刷新缓存操作失败了，这个异步回放的操作就可以补救。

### 缓存不一致的场景

缓存刷新策略：更新数据库后删除缓存。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/cache/cache-coherence_incoherence-01.drawio.png">
</div>

当两个请求分别由 service a 和 service b 处理时，如果 service a 的第三步比 service b 的第二步慢，缓存最终就会是错误的。

缓存刷新策略：更新数据库后更新缓存。

<div style="text-align: center; margin: 5px auto">
<img src="/image/computer-science/cache/cache-coherence_incoherence-02.drawio.png">
</div>

当两个请求分别由 service a 和 service b 处理时，如果 service a 的第三步比 service b 的第二步慢，缓存最终就会是错误的。

### 解决不一致的方案

将上面两个场景的 service a 的第三步，改成添加缓存。添加缓存的意思是，如果缓存存在，就什么都不做；如果缓存不存在，就添加。

这样就可以保证 service a 的第三步拿到的过期的缓存不会把后来正确的缓存刷掉。即使失败了，还可以被异步回放补救。

### reference（参考）

- 极客时间：Go 进阶训练营
