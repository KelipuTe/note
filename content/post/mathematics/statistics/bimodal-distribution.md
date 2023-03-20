---
draft: false
date: 2022-06-01 08:00:00 +0800
lastmod: 2022-06-01 08:00:00 +0800
title: "双峰分布"
summary: "双峰分布"
toc: true

categories:
- data-structure(数据结构)

tags:
- mathematics(数学)
- statistics(统计学)
---
### 双峰分布（bimodal distribution）

X 轴定义为一个请求需要消耗的时间，Y 轴定义为请求出现的次数。

对请求耗时进行采样，最终大量的请求的耗时会集中落在两个数值附近（正常耗时、超时）。

<div style="text-align: center; margin: 5px auto">
<img src="/image/mathematics/statistics/bimodal-distribution.drawio.png">
</div>

95%（99%）的请求的耗时是在合理范围内的，5%（1%）的请求可能永远不会完成。

这种少量的长耗时的请求会长时间占用系统资源，最终有可能让系统 OOM（Out Of Memory）或者拖垮系统。

### 参考

- 极客时间：Go 进阶训练营