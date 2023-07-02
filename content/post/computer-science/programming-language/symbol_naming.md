---
draft: false
date: 2022-11-07 08:00:00 +0800
title: "符号命名"
summary: "关于我的个人项目中的符号命名规则；"
toc: true

categories:
  - programming-language(编程语言)

tags:
  - computer-science(计算机科学)
  - programming-language(编程语言)
---

## 前言

实际开发中，符号命名规则应该和团队的风格保持统一，这套符号命名规则是我的个人喜好。
如果访客您非要说不符合哪里哪里的规范，怎么怎么有问题，那么访客您说的都对。

## 正文

实际的项目中有可能有出入，因为，前后改过好几次，同时，不排除以后还会继续改。

### 符号命名

我的符号（变量名、方法名）命名方式和主流的命名方式比起来有一些奇怪。区别主要在于各种各样标记类型的前缀。
声明的变量名和方法名、定义的类型名，都会带前缀。加前缀的目的，主要是为一些特殊的类型作标记，编码过程中引起重视。

#### 前缀都是什么意思

- a5，array，数组；
- c7，channel，管道；
- c5，const，常量；
- d9，dimension，维度；
- f8，function，函数、方法；
- g6，global，全局；
- i9，interface，接口；
- l5，local，局部；
- m3，map，map；
- p7，pointer，指针；
- s5，slice，切片；
- s6，struct，结构体；
- t4，temp，临时；

#### 前缀用法举例

`var a5UserID []int`，声明一个切片，切片的内容是 int。

`struct S6User {}`、`type S6User struct{}`，声明一个结构体。

`var m3s6user map[string]S6User`，声明一个 map，map 的内容是结构体。

`var m3s6p7user map[string]*S6User`，声明一个 map，map 的内容是指向结构体的指针。

`int t4status`，声明一个 int 临时变量。

#### 大写、小写、中划线、下划线

- 程序代码中的符号命名全部都用驼峰（camel case）。
- 缓存命名、数据库命名全部都用蛇形（snake case）。
- 命名目录时，两个或以上的单词用中划线连接。

- 如果变量需要对外开放（声明为 public），则前缀全部为大写。
- 如果变量不需要对外开放（声明为 protected、private），则前缀全部为小写。
- 如果变量名是一个单词，则单词首字母用小写（`var username string`）。
- 如果变量名是一个单词而且有前缀，则单词首字母用小写（`var s5username []string`）。
- 如果变量名是两个或以上的单词，则第一个单词首字母用小写，后面的单词首字母用大写（`var userID int`）。
- 如果变量名是两个或以上的单词而且有前缀，则单词首字母用大写（`var s5UserID []int`）。

- golang 项目，目录里的 "hkn" 后缀，用于回避关键字，比如：maphkn。
- golang 项目，import 引入时，两个或以上的单词用下划线连接。

### 版本号

我主要是各种原型验证项目会用到版本号，格式一般是 x.y，暂时达不到需要使用 x.y.z 这样的规模。
版本号从 0 开始，以 2 为步长递增，一般不会超过 10，常见的是：v00、v20、v22、v40、v42。
