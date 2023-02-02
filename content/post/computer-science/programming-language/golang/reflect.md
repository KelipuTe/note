---
draft: true
date: 2022-09-03 08:00:00 +0800
lastmod: 2022-09-03 08:00:00 +0800
title: "Golang 的反射的使用"
summary: "反射的基本使用方式。"
toc: true

categories:
- golang

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- golang
- reflect(反射)
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> go version go1.19 windows/amd64

### 实例

在 Golang 中，一个实例（叫变量也行），它由类型信息和值信息组成。比如：`var int a = 8`。a 就是一个实例，它的类型信息就是 int，值信息就是 8。当然实际上没有这么简单，这里就是方便理解。

### 反射

在 Golang 中，和反射相关的 API 都在 reflect 包里面。其中最核心的 API 有两个：`reflect.Type` 和 `reflect.Value`。reflect 包有一个很强的假设：使用者知道他在干什么。比如使用者知道他在操作的变量是什么 `reflect.Kind` 的。所以在调用 API 之前一定要先读注释，确认什么样的情况下可以调用。如果调用的不对就会 panic。

reflect.Type 可以用来操作类型信息，类型信息只能读取。这没啥好说的，代码里定义了一个 int 实例，编译的时候这个实例肯定也是 int 的，总不能运行的时候能把这个实例的 int 类型给改了吧。reflect.Value 可以用来操作值信息，部分值信息可以修改。为啥是部分信息呢，这很好理解，比如，私有变量就应该不能改，能改的话，就破坏私有变量的定义了。

另外，通过 reflect.Value 可以获取 reflect.Type，反过来则不行。这很好理解，在计算机中数据都是存储在内存上的 0 和 1，既然 reflect.Value 可以操作值信息，那么它就必须知道，数据在哪里，有多长。如果没有类型信息，它就不知道要从内存上拿多长的数据。反过来肯定不行，只知道类型，不代表知道数据在哪。

### 操作结构体

操作结构体有两种情况：第一种直接就是结构体、第二种是结构体指针。

#### 读取结构体的类型信息和值信息

如果直接就是结构体：

- `reflect.TypeOf()`：得到类型信息（结构体）
- `reflect.TypeOf().Kind()`：得到具体类型（枚举：Struct）
- `reflect.ValueOf()`：得到值信息（结构体的值）
- `reflect.ValueOf().Type()`：得到类型信息（结构体）

如果是结构体指针，需要通过 `Elem()` 拿到指针指向的实例：

- `reflect.TypeOf()`：得到类型信息（指针）
- `reflect.TypeOf().Kind()`：得到具体类型（枚举：Pointer）
- `reflect.TypeOf().Elem()`：得到类型信息（结构体）
- `reflect.TypeOf().Elem().Kind()`：得到具体类型（枚举：Struct）
- `reflect.ValueOf()`：得到值信息（指针）
- `reflect.ValueOf().Type()`：得到类型信息（指针）
- `reflect.ValueOf().Elem()`：得到值信息（结构体）
- `reflect.ValueOf().Elem().Type()`：得到类型信息（结构体）

#### 读取结构体属性的类型信息和值信息

上面通过 `reflect.TypeOf()` 和 `reflect.ValueOf()` 拿到了结构体的类型信息和值信息。
