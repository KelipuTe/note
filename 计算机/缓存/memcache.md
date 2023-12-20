---
draft: true
date: 2022-06-01 08:00:00 +0800
lastmod: 2022-06-01 08:00:00 +0800
title: "Memcache"
summary: "Memcache"
toc: true

categories:
- cache(缓存)

tags:
- computer-science(计算机科学)
- cache(缓存)
- memcache
---

### Memcache

提供简单的 KV 存储，所有的数据都在内存中，不支持持久化。

value 的大小默认不能超过 1MB，超过会报错，这个限制可以通过启动参数调整。

### 线程模型

使用的是多线程模型。

处理大 value 的时候，不会因为某个线程处理大文本被堵住导致整体吞吐下降。

### 内存模型

使用了 slab（译文：厚块）的方式做内存管理。每个 slab 包含若干大小为 1MB 的内存页，每个内存页又被分割成多个 chunk（译文：厚块）。可以理解成一种内存池的设计。

slab 有很多种，同一个 slab 中的 chunk 是一样大的。chunk 的增长因子由启动参数 `-f` 调整，默认为 1.25。起始 chunk 大小为 48 byte。每个 slab 会预先申请一整块内存，切割成多个 chunk，形成一个空闲内存块列表。

存储数据时，可以通过二分查找，看看哪个 slab 可以放得下数据，然后从那个 slab 的空闲内存块列表找一个内存块把数据放进去。已经分配出去的内存块，会维护一个 LRU 链表。移除数据时，直接把内存块还到内存池的空闲内存块列表，不需要真的释放内存。

### 注意点

如果需要大量存储某一种大小差不多的数据。

- 建议调整参数来优化每一种 slab 增长的 ratio（比例）
- 建议设置 slab_automove（自动移动） 和 slab_reassign（再分配）

可以防止引发 LRU 淘汰。

内存被某一种大小的 slab 分完了，这时如果有大于这个 slab 大小的数据过来，由于没有合适的 chunk，就会触发 LRU 淘汰一部分数据空出内存。

### 缓存技巧

- 批量读取，批量写入
- flag 的使用：compress、encoding、large value 等
- 使用二进制协议

### reference（参考）

- 极客时间：Go 进阶训练营
