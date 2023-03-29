---
title: "string and struct memory layout（字符串和结构体的内存布局）"
date: 2022-07-05 08:00:00 +0800
tags: computer-science operating-system linux memory
comment: false
show_author_profile: true
show_subscribe: false
---

> 操作系统架构：x86_64 AMD Ryzen <br/>
> 操作系统版本：Ubuntu 2204

> 样例代码 <br/>
> demo-c/demo-in-linux/memory-layout/string.c <br/>
> demo-c/demo-in-linux/memory-layout/struct.c

### 字符串

```
> ./string

arr1Statically  value=0x7fffffffde50
arr1Statically  value=arr1Statically1
p1Statically    value=0x55555555603a
p1Statically    value=p1Statically1
p1Statically    value=0x55555555607e
p1Statically    value=p1Statically2
p1Dynamically   value=0x5555555596b0
p1Dynamically   value=p1Dynamically1
malloc_usable_size=24
```

进程内存信息：

```
      0x555555556000     0x555555557000     0x1000     0x2000  r--p   /mnt/hgfs/demo-c/demo-in-linux/memory-layout/string
      0x555555559000     0x55555557a000    0x21000        0x0  rw-p   [heap]
      0x7ffffffde000     0x7ffffffff000    0x21000        0x0  rw-p   [stack]
```

- `char arr1Statically[16] = "arr1Statically1";`，数组存储的字符串在栈区（内存可读写）
- `char *p1Statically = "p1Statically1";`，这一步赋值的时候，字符串其实存在常量区（内存只读），指针存储的是字符串的起始地址
- `p1Statically = "p1Statically2";`，这一步重新赋值的时候，字符串还是存在常量区（内存只读），只是把指针指向的地址换到新的字符串的起始地址
- `strcpy(p1Dynamically, "p1Dynamically1");`，这里动态申请了一块内存，然后把字符串拷贝进去，所以存储的字符串在堆区（内存可读写）

`malloc_usable_size=24`，表示未使用的空间还有 24 byte，但是申请的时候只申请了 16 byte，所以这里其实申请了 40 byte。 这个是编译器的操作，申请内存的时候多给的部分，在需要扩容的时候就可以直接用了，不需要再通过系统调用去申请，可以减少系统调用的次数。

### 结构体

```
> ./struct

sizeof(Person) value=16

sizeof(a) value=16
&a        value=0x7fffffffde50
&a.name   value=0x7fffffffde50
&a.age    value=0x7fffffffde58
&a.sex    value=0x7fffffffde5c

sizeof(p1b) value=8
p1b         value=0x5555555596d0
p1b->name   value=0x5555555596f0
p1b->age    value=0x5555555596d8
p1b->sex    value=0x5555555596dc
```

结构体 a 的大小为：8 + 4 + 1 = 13 byte，但是由于结构体对其规则，这里用 sizeof 计算出的大小会是 16 byte。

```
(gdb) p &a
$8 = (struct person *) 0x7fffffffde50
(gdb) p &a.name
$9 = (char **) 0x7fffffffde50
(gdb) p &a.age
$10 = (int *) 0x7fffffffde58
(gdb) p &a.sex
$11 = 0x7fffffffde5c "f"
```

a 直接声明为结构体变量。所以，`sizeof(a) value=16`，而且所有的数据都在栈空间。

```
(gdb) x/8xb &a.name
0x7fffffffde50:	0xb0	0x96	0x55	0x55	0x55	0x55	0x00	0x00
(gdb) x/8xb a.name
0x5555555596b0:	0x61	0x61	0x61	0x00	0x00	0x00	0x00	0x00
```

这里需要注意 name 字段是个指针，需要动态申请内存，动态申请的内存在堆空间。

```
(gdb) p p1b
$2 = (struct Person *) 0x5555555596d0
(gdb) p &p1b->name
$4 = (char **) 0x5555555596d0
(gdb) p &p1b->age
$5 = (int *) 0x5555555596d8
(gdb) p &p1b->sex
$6 = 0x5555555596dc "f"
```

p1b 声明为结构体指针，所以，`sizeof(p1b) value=8`，而且需要动态申请内存。

```
(gdb) x/8xb &p1b->name
0x5555555596d0:	0xf0	0x96	0x55	0x55	0x55	0x55	0x00	0x00
(gdb) x/8xb p1b->name
0x5555555596f0:	0x62	0x62	0x62	0x00	0x00	0x00	0x00	0x00
```

p1b 的 name 字段同理，需要动态申请内存。与 a 不同的是，p1b 的 name 字段本身也在堆空间。

### reference（参考）

- 北风之神（微信：Le-studyg）
  - 高级课程
