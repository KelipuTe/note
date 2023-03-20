---
title: "human-computer interaction（人机交互）"
create_date: 2021-11-10 08:00:00 +0800
date: 2021-11-10 08:00:00 +0800
tags: computer-science graphics
comment: false
show_author_profile: true
show_subscribe: false
---

- command line interface（命令行界面）
  - 电传交互
  - teletype machine（电传打字机）
- 屏幕
  - text-based computer games（文字游戏）
- 2D 图形
  - CTR（cathode ray tubes、阴极射线管）
  - vector scanning（矢量扫描）
  - raster scanning（光栅扫描）
  - character generator（字符生成器）
    - dot matrix pattern（点阵图案）
    - screen buffer（屏幕缓冲区）
  - graphics cards（显卡）
- vector graphics（矢量图形）
- interactive graphical interface（交互式图形界面）
  - CAD（computer-aided design、计算机辅助设计）
- 显示器
  - pixel（像素）
  - bitmapped displays（位图显示）
  - frame buffer（帧缓冲区）
  - VRAM（video RAM、图象存储器）
  - LCD（liquid crystal display、液晶显示器）
- CGI（computer graphics interface、计算机图形接口）
- GUI（graphical user interface、图形用户界面、图形用户接口）
  - event-driven programming（事件驱动编程）
  - desktop metaphor（桌面隐喻）
- 3D 图形
  - 3-dimensional projection（3D 投影）
  - wireframe rendering（线框渲染）
  - orthographic projection（正交投影）
  - perspective projection（透视投射）
  - polygons（多边形）
  - mesh（网格）
  - scanline rendering（扫描线算法）
    - anti-aliasing（抗锯齿）
  - occlusion（遮挡）
    - painter's algorithm（画家算法）
    - z buffering（深度缓冲）
  - back face culling（背面剔除）
  - shading（明暗处理）
    - surface normal（表面法线）
    - flat shading（平面着色）
      - gourand shading（高洛德着色）
      - phong shading（冯氏着色）
  - texture（纹理）
    - texture mapping（纹理映射）
- GPU（graphics processing unit、图形处理单元）

### 2D 图形

- 矢量扫描：引导电子束直接绘制出形状
- 光栅扫描：按固定路径一行一行绘制

### 3D 图形

####

- 3D 投影：通过图形算法，把 3D 坐标平"拍平"，显示到 2D 屏幕上。
- 线框渲染：当所有的点都从 3D 转成 2D 之后，就可以用画 2D 线段的函数，连接这些点。

####

- 多边形：在 3D 图形学中，三角形也叫多边形。
- 网格：一堆多边形的集合。

####

- 画家算法：把多边形从远到近排序，从远到近渲染，想画家画画一样。
- 深度缓冲：通过给多边形设置深度值，来判断哪个多边形的像素在上面。当两个图形重叠的部分深度值一样的时候，最终显示哪个，是看算法怎么算浮点数的。因为不可控，所以就会出现重叠的部分一直闪的情况。

####

- 扫描线算法：这里以三角形为例。先铺一层像素网络，然后找出三角形最大纵坐标和最小纵坐标，只需要在这个范围内工作。然后算法从上往下，一次处理一行，计算每一行和三角形相交的两个点，然后填充这两个点之间的像素。当然，这样的三角形比较丑，边缘满是锯齿。
- 抗锯齿：使用扫描线算法时，每个像素不都涂成一样的颜色。可以通过判断多边形切过像素的程度，来调整颜色。如果像素在多边形内部，就直接涂色；如果多边形划过像素，颜色就浅一点。这种边缘羽化的效果，看着更舒服一些。

####

- 纹理：在图形学中指外观，而不是手感。
