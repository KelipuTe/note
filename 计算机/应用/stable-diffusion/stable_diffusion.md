---
draft: false
date: 2023-04-01 08:00:00 +0800
lastmod: 2023-04-03 08:00:00 +0800
title: "Stable Diffusion"
summary: "Stable Diffusion；Stable Diffusion web UI；Tag；ControlNet；"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- artificial-intelligence(人工智能)
- stable-diffusion
---
## 正文

### Stable Diffusion + Stable Diffusion web UI + 启动器

- {bilibili}/{Aniraiden}/[【AI制图教程】5分钟从零极速教会你用AI出图，职业美术实际使用经验技巧](https://www.bilibili.com/video/BV1pL411S72X/)
- {bilibili}/{秋葉aaaki}/[【AI绘画】Stable Diffusion 最终版 无需额外下载安装！可更新✓ 训练✓ 汉化✓ 提供7G模型 NovelAI](https://www.bilibili.com/video/BV17d4y1C73R/)
- {bilibili}/{秋葉aaaki}/[【AI绘画】启动器正式发布！一键启动/修复/更新/模型下载管理全支持！](https://www.bilibili.com/video/BV1ne4y1V7QU/)

大佬提供了整合包和启动器。整合包只有一个压缩包。启动器有两个文件，一个是启动器的压缩包，一个是启动器运行依赖。先安装启动器运行依赖。

整合包的压缩包和启动器的压缩包直接解压即可。这里假设，整合包解压之后的目录路径是 {Stable Diffusion}，方便下面的说明。

解压完成后，把启动器目录下的所有文件移动到 {Stable Diffusion} 目录覆盖。然后，就可以使用 {Stable Diffusion} 目录下的 **A启动器.exe** 启动了。

如果没有什么特别的需要，打开启动器后，直接点击右下角的一键启动即可。点击一键启动后，会弹出一个控制台，这个就是程序的本体，不要关掉了。

等待程序启动，看见控制台里输出 `Running on local URL:  http://127.0.0.1:7860` 的时候，就表示启动成功了。

这个时候，应该会自动在浏览器里打开 http://127.0.0.1:7860。如果没有打开的话，可以手动打开浏览器，然后访问 http://127.0.0.1:7860。这里假设，这个界面叫 {web UI}，方便下面的说明。

### 模型

模型，一般去 [civitai](https://civitai.com) 下载。常用的一般是 CheckPoint 大模型和 LoRA 小模型。

#### CheckPoint

把下载下来的模型放到 {Stable Diffusion}\models\Stable-diffusion\ 目录。

{web UI} 界面左上角，Stable Diffusion 模型(ckpt)，在下拉列表里选择需要启用的模型。

#### LoRA

- {bilibili}/{秋葉aaaki}/[【AI绘画】全新的微调模型！LoRA模型使用教程 插件安装 NovelAI](https://www.bilibili.com/video/BV1Py4y1d7eJ/)

把下载下来的模型 {Stable Diffusion}\extensions\sd-webui-additional-networks\models\lora\ 目录。

正常的写 Tag。然后，设置 "可选附加网络(LoRA 插件)" 选项。勾选 "启用"，然后，选择需要的附加网络类型，设置权重。

### Tag 语法

- ','：用于分隔不同的关键词。比如：white hair,red eyes。
- '|'：用于等比例混合。在 web UI 才可以用。比如：white|black hair。
- "(tag:权重数值)"：用于设置 tag 的权重。权重数值 0.1~100。
- "(tag)"、"((tag))"：用于增加权重。一层小括号，权重增加 1.1 倍。
- "\[tag\]"、"\[\[tag\]\]"：用于减少权重。一层中括号，权重减少 1.1 倍。
- "\[tag1:tag2:数字\]"：用于渐变。数字大于 1 表示，第{数字}步之前 tag 1，第{数字}步之后 tag 2。数字小于 1 表示，总步数百分之{数字}之前 tag 1，总步数百分之{数字}之后 tag 2。比如：\[white hair:black hair:5\]。
- "\[tag1|tag2\]"：用于交替。比如：\[white hair|black hair\]。

### Prompt（提示词）

Tag 不是越多越好，控制在 100 个以内。只写最关键的 Tag，别的 Tag 有需要的时候在加。注意 Tag 之间的冲突，比如：全身+上半身、长腿+短腿、等。

注意 Tag 的顺序，基本按照画面从上到下的顺序。同类的 Tag 里面，越关键的 Tag，越往前放。但是，有的时候，需要越级调整；有的时候，LoRA 模型有 Tag 要求。

### Negative prompt（反向提示词）

反向提示词就是不想出现在画面里的 Tag。这个也不是越多越好，画面有问题的时候在加。

### 常用 Negative prompt

EasyNegative,
(worst quality, low quality:1.4),
(poorly drawn face:1.4),
(extra limbs:1.35),
(malformed hands:1.4),(poorly drawn hands:1.4),(mutated fingers:1.4),

### 异常

#### 报错 1

```
NansException: A tensor with all NaNs was produced in VAE. This could be because there's not enough precision to represent the picture. Try adding --no-half-vae commandline argument to fix this. Use --disable-nan-check commandline argument to disable this check.
```

打开启动器，在左侧菜单栏找到高级选项，在高级选项界面，勾选 "不使用半精度 VAE(--no-half-vae)" 和 "关闭数值溢出检查(--disable-nan-check)"。

### ControlNet

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

正常的写 Tag。然后，设置 "扩散控制网络(ControlNet)" 选项。勾选 "启用"，如果显存低于 8G，还需要勾选 "低显存优化"。然后，选择预处理器和模型，这两个要匹配着选。

- 预处理器选 "canny"，模型选 "control_canny"，实现边缘检测，达到上色的效果。
- 预处理器选 "depth"，模型选 "control_depth"，实现深度图，达到指定的画面结构。
- 预处理器选 "hed"，模型选 "control_hed"，实现 hed 边缘检测，没有 canny 那么精准的控制，AI 可以自由发挥。
- 预处理器选 "openpose"，模型选 "control_openpose"，实现人体动作控制。可以用其他软件做出骨骼图。
- 还有很多其他的。

比如，预处理器选 "openpose 姿态及手部检测"，模型选 "control_openpost"。然后，给 ControlNet 一张图片（最好是真人的）。然后，就可以开始生成图片了。

预处理器会参考给的图片生成对应的生成条件图，ControlNet 会用生成条件图去指导 AI 生成图片。这里是可以自己造生成条件图，然后，直接给 ControlNet 的。

### 个人比较喜欢的模型

#### CheckPoint 模型

| 名字 | 平台 | id | 链接 | 描述 |
| --- | --- | --- | --- | --- |
| AbyssOrangeMix2 - Hardcore | civitai | 4451 | https://civitai.com/models/4451/abyssorangemix2-hardcore | 二次元、涩涩 |
| Counterfeit-V2.5 | civitai | 4468 | https://civitai.com/models/4468/counterfeit-v25 | 二次元 |
| AnyHentai | civitai | 5706 | https://civitai.com/models/5706/anyhentai | 二次元、涩涩 |
| ChilloutMix | civitai | 6424 | https://civitai.com/models/6424/chilloutmix | 写实、涩涩 |
| Cetus-Mix | civitai | 6755 | https://civitai.com/models/6755/cetus-mix | 二次元 |
| MeinaMix | civitai | 7240 | https://civitai.com/models/7240/meinamix | 二次元、涩涩 |
| AbyssOrangeMix3 (AOM3) | civitai | 9942 | https://civitai.com/models/9942/abyssorangemix3-aom3 | 二次元、涩涩 |
| MeinaHentai | civitai | 12606 | https://civitai.com/models/12606/meinahentai | 二次元、涩涩 |
| AOAOKO \[PVC Style Model\] | civitai | 15509 | https://civitai.com/models/15509/aoaoko-pvc-style-model | 二次元、PVC 模型 |

#### LoRA 模型

| 名字 | 平台 | id | 链接 | 描述 | 额外说明 |
| --- | --- | --- | --- | --- | --- |
| X-Ray Hentai 2.5 | civitai | 3938 | https://civitai.com/models/3938/x-ray-hentai-25 | 二次元、x-射线透视 | Tag：x-ray、x-ray view、cross-section。采样方法：Euler、Euler A。 |
| Eye - LoRa | civitai | 5529 | https://civitai.com/models/5529/eye-lora | 二次元、眼睛 | Tag：loraeyes。 |
| 明日方舟-年 Arknights-Nian | civitai | 8185 | https://civitai.com/models/8185/arknights-nian | 明日方舟-年 | Tag：origen(普通年)、china dress(旗袍年)。 |
| POV Doggystyle LoRA \[1 MB\] | civitai | 8723 | https://civitai.com/models/8723/pov-doggystyle-lora-1-mb | 二次元、后背位 | Tag：`<lora:POVDoggy:0.9>`。权重 0.9~1.0。Tag penis，如果有一般是浅入，如果没有一般是深入。 |
| breastInClass: Better Bodies | civitai | 9025 | https://civitai.com/models/9025/breastinclass-better-bodies | 写实、身体 | Tag：`<lora:breastinclassbetter_v141:0.5>` |
| Innies: Better vulva | civitai | 10364 | https://civitai.com/models/10364/innies-better-vulva | 写实、阴部 | Tag：`<lora:inniesbettervaginas_v11:1.0>` |
| \[NSFW\]Tentacles LoRA \| 触手 | civitai | 11886 | https://civitai.com/models/11886/nsfwtentacles-lora-or | 二次元、触手 | Tag：tentacles。 |
| H&K HK416 LoRA | civitai | 12519 | https://civitai.com/models/12519/handk-hk416-lora | HK416 | Tag：gun、weapons、holding weapon、assault rifle。 |
| 站立后背位/立ちバック/standing doggystyle | civitai | 12682 | https://civitai.com/models/12682/standing-doggystyle | 二次元、后背位 | 权重 0.5~0.7。共通的 tag 为 1girl,1boy,sex。配合 from normal 可以生成后背的视角、配合 from front 可以生成正对的视角、配合 from side 可以生成侧面的视角、配合 from behind,pov 可以生成后背的 pov 视角。 |
| Doggystyle from side view | civitai | 12961 | https://civitai.com/models/12961/doggystyle-from-side-view | 二次元、后背位、侧面的视角 | 采样方法：DPM SDE Karras。 |
| Corruption/悪堕ち/恶堕 | civitai | 17610 | https://civitai.com/models/17610/corruption | 二次元、恶堕 | Tag：corruption、empty eyes、half-closed eyes、evil smile、no pupils、crazy smile。权重 0.3~0.7。 |
| Incoming hug/kiss | civitai | 21388 | https://civitai.com/models/21388/incoming-hugkiss | 二次元、抱过来、亲过来 | Tag：incoming hug(抱过来)、incoming kiss(亲过来)。Nagative：EasyNegative、bad-hands-5。 |
| Murky's After Sex Lying LoRA | civitai | 18194 | https://civitai.com/models/18194/murkys-after-sex-lying-lora | 二次元、事后 | Tag：after sex。 |

## 程序本体

- {github}/[CompVis/stable-diffusion](https://github.com/CompVis/stable-diffusion)
- {github}/[AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui)
- {github}/[Mikubill/sd-webui-controlnet](https://github.com/Mikubill/sd-webui-controlnet)
