---
draft: true
date: 2022-06-01 08:00:00 +0800
lastmod: 2022-06-01 08:00:00 +0800
title: "长尾效应"
summary: "长尾效应"
toc: true

categories:
- data-structure(数据结构)

tags:
- mathematics(数学)
- statistics(统计学)
---

### 长尾效应（Long Tail Effect）

X 轴定义为一个请求需要消耗的内存，Y 轴定义为请求出现的次数。

x0 左边的认为是内存消耗正常的请求，x0 右边的认为是内存消耗异常的请求。

<div style="text-align: center; margin: 5px auto">
<img src="/image/mathematics/statistics/long-tail-effect.drawio.png">
</div>

存在一种可能性 x0 ，让 x0 右边少量的异常请求消耗的内存总和，比 x0 左边大量的正常请求消耗的内存总和大。

这种少量的内存消耗异常的请求会大量占用系统资源，最终有可能让系统 OOM（Out Of Memory）或者拖垮系统。

### 参考

- 极客时间：Go 进阶训练营