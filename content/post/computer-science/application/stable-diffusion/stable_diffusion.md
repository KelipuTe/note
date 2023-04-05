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

#### 个人比较喜欢的 CheckPoint 模型

##### 4468

{civitai}/[Counterfeit-V2.5](https://civitai.com/models/4468/counterfeit-v25)

二次元。

##### 6424

{civitai}/[ChilloutMix](https://civitai.com/models/6424/chilloutmix)

写实、涩涩。

##### 6755

{civitai}/[Cetus-Mix](https://civitai.com/models/6755/cetus-mix)

二次元。

##### 9942

{civitai}/[AbyssOrangeMix3 (AOM3)](https://civitai.com/models/9942/abyssorangemix3-aom3)

二次元、涩涩。

#### LoRA

- {bilibili}/{秋葉aaaki}/[【AI绘画】全新的微调模型！LoRA模型使用教程 插件安装 NovelAI](https://www.bilibili.com/video/BV1Py4y1d7eJ/)

把下载下来的模型 {Stable Diffusion}\extensions\sd-webui-additional-networks\models\lora\ 目录。

正常的写 Tag。然后，设置 "可选附加网络(LoRA 插件)" 选项。勾选 "启用"，然后，选择需要的附加网络类型，设置权重。

#### 个人比较喜欢的 LoRA 模型

##### 3938

{civitai}/[X-Ray Hentai 2.5](https://civitai.com/models/3938/x-ray-hentai-25)

二次元、x-射线视图。Tag：x-ray、x-ray view、cross-section、vaginal、uterus、cervix、penis hitting uterus、internal cumshot。采样方法：Euler、Euler A。

样例 1：1girl,1boy,cross-section, cum, cum_in_pussy, ejaculation,internal_cumshot, penis, pussy, sex uterus, vaginal, busty, curvaceous, grin

样例 2：1boy, 1girl, ahegao, big breasts, wide hips, nun, 4k, crop-top, sex, loose clothing, sex from behind, muscular male, fucked silly, cross-section, x-ray, internal cumshot, ejaculation, penis hitting uterus, uterus,

##### 5529

{civitai}/[Eye - LoRa](https://civitai.com/models/5529/eye-lora)

二次元、眼睛。Tag：loraeyes。

##### 8185

{civitai}/[明日方舟-年 Arknights-Nian](https://civitai.com/models/8185/arknights-nian)

明日方舟、年。Tag：origen(普通年)、china dress(旗袍年)。

##### 8723

{civitai}/[POV Doggystyle LoRA \[1 MB\]](https://civitai.com/models/8723/pov-doggystyle-lora-1-mb)

二次元、后背位。Tag：`<lora:POVDoggy:0.9>,1girl,1boy,penis(可选),doggystyle,from behind`。权重 0.9~1.0。

penis，如果有一般是浅入，如果没有一般是深入。

pov hands, ass grab, deep skin (Good results)

facing away, nude, implied sex (Good results)

pov hands, spread anus, spread ass, deep skin, hands on ass (50/50 - Okay results.)

##### 9025

{civitai}/[breastInClass: Better Bodies](https://civitai.com/models/9025/breastinclass-better-bodies)

写实、身体。Tag：`<lora:breastinclassbetter_v141:0.5>`

##### 10364

{civitai}/[Innies: Better vulva](https://civitai.com/models/10364/innies-better-vulva)

写实、阴部。Tag：`<lora:inniesbettervaginas_v11:1.0>`

##### 11886

{civitai}/[\[NSFW\]Tentacles LoRA | 触手](https://civitai.com/models/11886/nsfwtentacles-lora-or)

二次元、触手。Tag：tentacles。

##### 12519

{civitai}/[H&K HK416 LoRA](https://civitai.com/models/12519/handk-hk416-lora)

HK416。Tag：gun、weapons、holding weapon、assault rifle。

##### 12682

{civitai}/[站立后背位/立ちバック/standing doggystyle](https://civitai.com/models/12682/standing-doggystyle)

二次元、后背位。Tag：`1girl,1boy,sex`。权重 0.5~0.7。

共通的 tag 为 1girl,1boy,sex。配合 from normal 可以生成后背的视角、配合 from front 可以生成正对的视角、配合 from side 可以生成侧面的视角、配合 sex from behind,pov 可以生成后背的 pov 视角。配合 breast grab,grabbing from behind 可以实现从后面抓胸。


##### 12961

{civitai}/[Doggystyle from side view](https://civitai.com/models/12961/doggystyle-from-side-view)

id=12961。二次元、后背位、侧面的视角。

(cum on body,cum in pussy,(1boy,1girl),sex from behind,implied sex,doggystyle),put it at beginning willapparently reduce the probability of this

generate another image or using EasyNegative and ng_deepnegative isa efficient way to improve it.

change sampler to DPM SDE karras

put it at models\Lora,and add this lora to your prompt


##### 17610

{civitai}/[Corruption/悪堕ち/恶堕](https://civitai.com/models/17610/corruption)

id=17610。二次元、恶堕。Tag：corruption、empty eyes、half-closed eyes、evil smile、no pupils、crazy smile。权重 0.3~0.7。

##### 21388

{civitai}/[Incoming hug/kiss](https://civitai.com/models/21388/incoming-hugkiss)

id=21388。二次元、抱过来、亲过来。Tag：incoming hug、incoming kiss。Nagative：EasyNegative、bad-hands-5。

##### 18194

{civitai}/[Murky's After Sex Lying LoRA](https://civitai.com/models/18194/murkys-after-sex-lying-lora)

id=18194。二次元、事后。

after sex, cum, lying, cumdrip, ass, on stomach, on back, fucked silly, sweat, cum pool, bukkake, trembling

### Tag

rolling eyes(翻白眼)、no panties(没有内裤)、mature female(成熟女性)

nuns clothing(修女服装)、ahegao(ahe颜)、tongue out(伸出舌头)

x-ray(x-射线)、x-ray view(x-射线视图)、cross-section(截面图)

busty(丰满的)、curvaceous(曲线优美的)、grin(咧嘴笑)

ejaculation(射精)

wide(宽大的) hips(臀部)

nun(修女)

looking at viewer(看向观察者)

waist(腰)

fucked silly

ass focus(聚焦)

facing away(朝向远处)

doggystyle(后背位)

implied sex(隐性性行为)

loose clothing(宽松的衣服)

spread(扩张，扩散)

muscular(肌肉发达的) male(男性)

crop-top(圆领衫)

penis(阴茎)、uterus sex

vaginal(阴道)、uterus(子宫)、cervix(子宫颈)、hitting uterus(撞击子宫)

#### 语法

- ','：用于分隔不同的关键词。比如：white hair,red eyes。
- '|'：用于等比例混合。在 web UI 才可以用。比如：white|black hair。
- "(tag,权重数值)"：用于设置 tag 的权重。权重数值 0.1~100。
- "(tag)"、"((tag))"：用于增加权重。一层小括号，权重增加 1.1 倍。
- "\[tag\]"、"\[\[tag\]\]"：用于减少权重。一层中括号，权重减少 1.1 倍。
- "\[tag1:tag2:数字\]"：用于渐变。数字大于 1 表示，第{数字}步之前 tag 1，第{数字}步之后 tag 2。数字小于 1 表示，总步数百分之{数字}之前 tag 1，总步数百分之{数字}之后 tag 2。比如：\[white hair:black hair:5\]。
- "\[tag1|tag2\]"：用于交替。比如：\[white hair|black hair\]。

#### Prompt（提示词）

Tag 不是越多越好，控制在 100 个以内。只写最关键的 Tag，别的 TAg 有需要的时候在加。

注意 Tag 的顺序，基本按照下面子标题的顺序。同类里面，越关键的词，越往前放。但是，有的时候需要越级调整，有的时候 LoRA 模型有 Tag 要求。

注意 Tag 之间的冲突，比如：全身+上半身、长腿+短腿、等。

##### 画质

masterpiece(杰作)、ultra-detailed(超详细的)、best quality(品质)、high highres(高分辨率)、

best illumination(照明)、best shadow(阴影)、high contrast(对比度)

##### 风格

anime(动画)、portrait(画像)、illustration(插图)、game cg、wallpaper(壁纸)、3d、

upper body(上半身)、full body(全身)、darkness(黑暗)

extremely(极度的) detailed(有细节的) CG unity(统一) 8k wallpaper



主题：

1girl、2girl、demon(恶魔)、succubus(梦魔)

主题的细节（从上到下）：

- 头发：braids(辫子)
- 头：perfect face(脸)、beautiful detailed(有细节的) face、beautiful detailed eyes、tears(眼泪)、fake animal ears(假的动物耳朵)、rabbit ears(兔耳朵)
- 颈：choker(颈圈)
- 衣服：lite armor(轻型装甲),heavy armor(重型装甲)、nude(裸体)、torn clothes(撕裂的衣服)、tentacles(触手)、tentacle sex、
- 肩：
- 臂：sleeveless(无袖)
- 手：perfect hands(手)、perfect fingers(手指)、wrist cuffs(腕带)
- 胸：small,medium,big breasts(胸部)、
- 腹：navel(肚脐)、under-stomach tattoo(肚脐下的纹身)、uterus tattoo(子宫位置的纹身)、crotch tattoo(下腹至私处之间的纹身)、tramp stamp(后腰的纹身)
- 阴：pussy(阴部)、pubic hair(阴毛)、pussy juice(阴部汁液)、pussy juice trail(阴部汁液痕迹)、pussy juice stain(阴部汁液污渍)、artificial vagina(人造阴部)、ass(屁股)、anus(肛门)
- 腿：perfect legs(腿)、pantyhose(连裤袜)
- 脚：feet(脚)、toes(脚趾)、soles(脚底)

主题的表情：

主题的姿势：

- 基础姿势：standing(站立)、hug each other(相互拥抱)
- 头：close one's eyes(闭上眼睛)、close one eye(闭上一只眼睛)、look at viewer(看着观众)
- 臂：cast a spell(施法)
- 腰：
- 阴：presenting(呈现)、pussy peek(阴部窥视)、spread pussy(掰开阴部)、internal cumshot(内射精液)、cum(精液、射精)、cum in pussy(射进阴部)、cum on body(射身体上)
- 腿：spread legs(张开双腿)
- 复合动作

背景：室内场景、室外场景、天气

其他：nsfw、hentai

#### Negative prompt（反向提示词）

low picture anime(低画质动画)、blurry(模糊的) image、low resolution(分辨率)

missing face、 missing body、missing legs,  bad hands,worst quality, low quality, monochrome(单色), zombie,text,logo,card,

malformed mutated,morbid,error,mutated,more than 2 thighs,malformed limbs,poorly drawn,poorly drawn hands,mutilated,missing fngers,fused fingers,more than 2 nipples,extra legs,disfigured,multiple breasts,bad face,fused anus,three arms,missing arms,missing limb,Missing limbs,missing fingers,malformed,mutated hands and fingers ,limb,mutated hands and fingers,cloned face,worstquality,low quality,long body,long neck,missing fingers,missing arms,bad hands,long neck,lowres,extra digit,jpeg artifacts,bad anatomy,signature,fewer digits,Humpbacked,cropped,watermark,text,worst quality,username,error,blurry,low quality,normal quality,bad anatomy disfigured malformed mutated,extra limbs,too many fingers,bad hands,mutated hands,bad anatomy,bad proportions,large breasts,too long legs,wrong colors,(Depth of field),big breasts,logo,Mosaic,hat,(twintails),(worst quality, low quality:1.4), logo, watermark, artist name,

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

## 程序本体

- {github}/[CompVis/stable-diffusion](https://github.com/CompVis/stable-diffusion)
- {github}/[AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui)
- {github}/[Mikubill/sd-webui-controlnet](https://github.com/Mikubill/sd-webui-controlnet)
