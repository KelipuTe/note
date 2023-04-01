---
draft: false
date: 2023-04-01 08:00:00 +0800
lastmod: 2023-04-01 08:00:00 +0800
title: "Stable Diffusion"
summary: "Stable Diffusion；Stable Diffusion web UI；"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- artificial-intelligence(人工智能)
- stable-diffusion
---
## 正文

### 视频教程

- {bilibili}/{Aniraiden}/[【AI制图教程】5分钟从零极速教会你用AI出图，职业美术实际使用经验技巧](https://www.bilibili.com/video/BV1pL411S72X/)
- {bilibili}/{秋葉aaaki}/[【AI绘画】Stable Diffusion 最终版 无需额外下载安装！可更新✓ 训练✓ 汉化✓ 提供7G模型 NovelAI](https://www.bilibili.com/video/BV17d4y1C73R/)
- {bilibili}/{秋葉aaaki}/[【AI绘画】启动器正式发布！一键启动/修复/更新/模型下载管理全支持！](https://www.bilibili.com/video/BV1ne4y1V7QU/)

大佬提供了整合包和启动器。整合包只有一个压缩包。启动器有两个文件，一个是启动器的压缩包，一个是启动器运行依赖。先安装启动器运行依赖。

整合包的压缩包和启动器的压缩包直接解压即可。这里假设，整合包解压之后的目录路径是 {Stable Diffusion}，方便下面的说明。

解压完成后，把启动器目录下的所有文件移动到 {Stable Diffusion} 目录覆盖。然后，就可以使用 {Stable Diffusion} 目录下的 **A启动器.exe** 启动了。

如果没有什么特别的需要，打开启动器后，直接点击右下角的一键启动即可。点击一键启动后，会弹出一个控制台，这个就是程序的本体，不要关掉了。

等待程序启动，看见控制台里输出 `Running on local URL:  http://127.0.0.1:7860` 的时候，就表示启动成功了。

这个时候，应该会自动在浏览器里打开 http://127.0.0.1:7860。如果没有打开的话，可以手动打开浏览器，然后访问 http://127.0.0.1:7860。这里假设，这个界面叫 {web UI}，方便下面的说明。

### controlnet

#### 安装

- {bilibili}/{秋葉aaaki}/[【AI绘画】完美控制画面！告别抽卡时代 人物动作控制/景深/线稿上色 Controlnet安装使用教程](https://www.bilibili.com/video/BV1Wo4y1i77v/)

大佬提供了整合包。这里假设，整合包解压之后的目录路径是 {controlnet}，方便下面的说明。

`{web UI} --> 扩展 --> 从网址安装`。在 "扩展的 git 仓库网址" 里，输入 https://jihulab.com/hunter0725/sd-webui-controlnet，然后点击下面的安装按钮。

等待安装，安装成功后，会在安装按钮的下面输出 `Installed into D:\stable-diffusion\extensions\sd-webui-controlnet. Use Installed tab to restart.`

把 {controlnet}\安装\openpose\ 目录下面的文件，放到 {Stable Diffusion}\extensions\sd-webui-controlnet\annotator\openpose\ 目录。

把 {controlnet}\安装\midas\ 目录下面的文件，放到 {Stable Diffusion}\extensions\sd-webui-controlnet\annotator\midas\ 目录。

把 {controlnet}\模型\ 目录下面的文件，放到 {Stable Diffusion}\extensions\sd-webui-controlnet\models\ 目录。

这些都搞定之后，`{web UI} --> 扩展 --> 已安装`。点击应用并重启用户界面按钮。重启完成后，应该就可以在文生图（别的也行）左下方的设置区域的最下面，看见 ControlNet 的选项。

#### 使用

正常的写 Tag。然后，设置 ControlNet 选项。勾选 "启用"，如果显存低于 8G，还需要勾选 "低显存优化"。然后，选择预处理器和模型，这两个要匹配着选。

- 预处理器选 "canny"，模型选 "control_canny"，实现边缘检测，达到上色的效果。
- 预处理器选 "depth"，模型选 "control_depth"，实现深度图，达到指定的画面结构。
- 预处理器选 "hed"，模型选 "control_hed"，实现 hed 边缘检测，没有 canny 那么精准的控制，AI 可以自由发挥。
- 预处理器选 "openpose"，模型选 "control_openpose"，实现人体动作控制。可以用其他软件做出骨骼图。
- 还有很多其他的。

比如，预处理器选 "openpose 姿态及手部检测"，模型选 "control_openpost"。然后，给 ControlNet 一张图片（最好是真人的）。然后，就可以开始生成图片了。

预处理器会参考给的图片生成对应的生成条件图，ControlNet 会用生成条件图去指导 AI 生成图片。这里是可以自己造生成条件图，然后，直接给 ControlNet 的。

## 程序本体

- {github}/[CompVis/stable-diffusion](https://github.com/CompVis/stable-diffusion)
- {github}/[AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui)
- {github}/[Mikubill/sd-webui-controlnet](https://github.com/Mikubill/sd-webui-controlnet)
