---
title: "compression（压缩）"
create_date: 2021-05-09 08:00:00 +0800
date: 2021-05-09 08:00:00 +0800
tags: computer-science algorithm
comment: false
show_author_profile: true
show_subscribe: false
---

- compression（压缩）
  - lossless compression（无损压缩）
    - run-length coding（RLE、游程编码）
    - dictionary coding（字典编码）
      - huffman coding（霍夫曼编码）
  - lossy compression（有损压缩）
    - perceptual coding（感知编码）
    - temporal redundancy（时间冗余）

### 压缩

压缩：用更少的空间来表示数据。

常见的压缩思路：消除冗余、使用更紧凑的表示形式。

#### 无损压缩

无损压缩：压缩前的数据和解压后得到的数据完全一致。

游程编码适合处理经常有连续数据的文件。

字典编码：用 map（字典）存储原始数据和紧凑代码的对应关系。字典也要占空间。

霍夫曼编码：根据数据出现的频率重新编码数据，使用短编码表示高频出现的数据。

#### 有损压缩

有损压缩：压缩时根据需要丢弃一些数据。

感知编码：丢弃人类无法感知的数据。依赖于人类的感知模型，模型来自 psychophysics（心理物理学）。

时间冗余：利用 inter-frame similarity（帧与帧之间的相似性），当图像或音频数据中出现相同数据时，不需要每一帧都保存完整的数据，可以只保存变了的部分。

高级的视频压缩算法会找出帧与帧之间相似的部分，然后通过简单的操作实现。比如，一个人在挥手的视频。压缩算法会把手看成一个整体，然后通过平移和旋转这个手的方式保存每一帧的数据。

压缩算法出错会导致图像出现撕裂或音频出现杂音，但是播放器依然会忠实的进行图像渲染或音频播放。

### 参考

- Crash Course Computer Science（计算机科学速成课）
  - [bilibili](https://www.bilibili.com/video/BV1EW411u7th)
  - [CrashCourse 字幕组](https://github.com/1c7/crash-course-computer-science-chinese)
  - [Youtube 原视频](https://www.youtube.com/playlist?list=PL8dPuuaLjXtNlUrzyH5r6jN9ulI)