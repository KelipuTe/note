---
title: "群星"
create_date: 2021-04-25 08:00:00 +0800
date: 2022-05-03 08:00:00 +0800
tags: game stellaris
comment: false
show_author_profile: true
show_subscribe: false
---

### 控制台

按下键盘`~`键呼出控制台。

开关命令表示输入命令式会切换命令生效的状态。

| 命令                                                         | 效果                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| help                                                         | 显示一个指令的帮助文档。                                     |
| debugtooltip                                                 | 开关，查看ID、效果等信息。                                   |
| add_opinion [target country id1] [target country id2] [amount] | 增加id1对id2的好感度(可以使用负数减少好感度)                 |
| event [event_id.target_id]                                   | 执行一个事件。[事件ID.目标ID/event_id.target_id]             |
| instant_build                                                | 开关，瞬间建造，建造费用为0，对AI同样生效。                  |
| invincible                                                   | 玩家的舰队将不会受到伤害。                                   |
| yesmen                                                       | 开关，AI总是同意玩家提出的条约。                             |
| finish_research                                              | 瞬间完成当前科技。                                           |
| research_all_technologies [数值1] [数值2]                    | 瞬间研究所有科技。<br/>数值1为事件武器和天灾武器和飞升武器，1为研究，0为不研究。<br/>数值2为循环科技等级。 |
| resource [resource_name] [数值]                              | 添加资源，数值不填，默认为5000。<br/>稀有文物--minor_artifacts；纳米机器人--nanites； |
| influence [数值]                                             | 添加影响力，默认为5000。                                     |
| grow_pops [数值]                                             | 添加成长中的人口至选中的星球，数值不填，默认为 1。           |
| unity [数值]                                                 | 添加凝聚力，默认为5000。                                     |
| alloys                                                       | 添加合金，默认为5000                                         |

#### 领袖特性

打开debugtooltip，只输入`add_trait_leader [leader_id]`不输入具体特性ID时，控制台会输出领袖特性列表里面可以用的特性的ID。同理，只输入`remove_trait_leader [leader_id]`不输入具体特性ID时，控制台会输出领袖当前拥有的特性的ID。

#### 事件

| 事件ID      | 描述     |
| ----------- | -------- |
| distar.50   | 泡泡获得 |
| graygoo.401 | 灰风获得 |

### MOD

#### MOD事件

| 事件ID          | 事件描述                     |
| --------------- | ---------------------------- |
| wsg.1100        | 分支选项，苍青和S |
| wsg.3066 | 神秘的AI飞船，大小姐出现 |
| wg_lady.1 | 堕落妈开漫展，大小姐改造 |
| wsg_shimakaze.1 | 奇怪的新舰娘，岛风获得            |
| wg_crisis.2  | 联合舰队事件开始 |

