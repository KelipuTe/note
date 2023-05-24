---
draft: true
create_date: 2021-04-25 08:00:00 +0800
date: 2022-05-03 08:00:00 +0800
title: "群星"
summary: "控制台；事件 ID；"
toc: true

categories:
- game

tags:
- game
---
## 控制台

按下键盘 `~` 键呼出控制台。

开关命令表示输入命令式会切换命令生效的状态。

| 命令                                                                   | 效果                                    |
|----------------------------------------------------------------------|---------------------------------------|
| help                                                                 | 显示一个指令的帮助文档。                          |
| debugtooltip                                                         | 开关，查看ID、效果等信息。                        |
| add_opinion \[target country id1\] \[target country id2\] \[amount\] | 增加 id1 对 id2 的好感度（可以使用负数减少好感度）        |
| event \[event_id.target_id\]                                         | 执行一个事件。[事件ID.目标ID/event_id.target_id] |
| instant_build                                                        | 开关，瞬间建造，建造费用为0，对AI同样生效。               |
| grow_pops \[数值\]                                                     | 添加成长中的人口至选中的星球，数值不填，默认为 1。            |

## 领袖特性

打开 debugtooltip，只输入 `add_trait_leader [leader_id]` 不输入具体特性 ID 时，控制台会输出领袖特性列表里面可以用的特性的 ID。

同理，只输入 `remove_trait_leader [leader_id]` 不输入具体特性 ID 时，控制台会输出领袖当前拥有的特性的 ID。

## 本体事件

| 事件ID         | 描述     |
|--------------|--------|
| distar.50    | 泡泡获得   |
| graygoo.401  | 灰风获得   |

## MOD事件

| 事件ID            | 事件描述          |
|-----------------|---------------|
| wsg.1100        | 分支选项，苍青和S     |
| wsg.3066        | 神秘的AI飞船，大小姐出现 |
| wg_lady.1       | 堕落妈开漫展，大小姐改造  |
| wsg_shimakaze.1 | 奇怪的新舰娘，岛风获得   |
| wg_crisis.2     | 联合舰队事件开始      |

