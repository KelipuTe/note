

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
