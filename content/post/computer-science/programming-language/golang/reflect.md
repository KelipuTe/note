---
draft: false
create_date: 2022-09-03 08:00:00 +0800
date: 2023-05-22 08:00:00 +0800
title: "Golang 反射的基本使用方式"
summary: "反射的基本使用方式；"
toc: true

categories:
- golang

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- golang
- reflect(反射)
---
## 前言

- CPU AMD64(x86_64)
- Windows 11 家庭版
- go version go1.19 windows/amd64

## 资料

- [{demo-golang}](https://github.com/KelipuTe/demo-golang)/demo/reflectkn/
- <a href="/drawio/computer-science/programming-language/golang/reflect.drawio.html">reflect.drawio.html</a>

## 正文

注意，笔记里的代码是伪代码，方便说明和区分用的。

### 实例

在 Golang 中，一个实例（叫变量也行），它由类型信息和值信息组成。

比如：`var int a = 8`。a 就是一个实例，它的类型信息就是 int，值信息就是 8。

当然实际上没有这么简单，这里就是方便理解。

### 反射

和反射相关的 API 都在 reflect 包里面。其中最核心的 API 有两个：reflect.Type() 和 reflect.Value()。

reflect 包有一个很强的假设，使用者知道他在干什么。比如，使用者知道他在操作的变量是什么类型的。

所以，在调用 API 之前一定要先读注释，确认什么样的情况下可以调用。如果调用的不对就会 panic。

reflect.Type() 可以用来操作类型信息，类型信息只能读取。比如，代码里定义了一个 int 实例，编译的时候这个实例肯定也是 int 的。

reflect.Value() 可以用来操作值信息，部分值信息可以修改。比如，私有变量就应该不能改，能改的话，就破坏私有变量的定义了。

另外，通过 reflect.Value() 可以获取 reflect.Type()，反过来则不行。

在计算机中数据都是存储在内存上的 0 和 1，既然 reflect.Value() 可以操作值信息，那么它就必须知道，数据在哪里，有多长。

如果没有类型信息，它就不知道要从内存上拿多长的数据。反过来则不行，只知道类型，不代表知道数据在哪。

### 基本类型

基本类型比较简单了。比如，int 类型。

reflect.TypeOf(int).Kind() 直接就能拿到 reflect.Int 类型。

reflect.ValueOf(int).Interface() 直接就能拿到 interface{} 类型的值。再做一次类型断言，就可以得到 int 类型的值了。

代码示例：**{demo-golang}/demo/reflectkn/int_test.c**

### array、slice、string

这几个类型不能直接用 reflect.ValueOf(array).Interface()。

要先通过 reflect.ValueOf(array).Len() 拿到长度。然后，通过 reflect.ValueOf(array).Index(i) 拿到每一个元素。

拿到每一个元素之后，就可以通过 Index(i).Interface() 拿到具体的值了。

如果是多维数组的话，Index(i).Interface() 会拿到多维数组的元素。比如，二维数组，这里拿到的就是一维数组。

代码示例：

- **{demo-golang}/demo/reflectkn/array_test.c**
- **{demo-golang}/demo/reflectkn/slice_test.c**
- **{demo-golang}/demo/reflectkn/string_test.c**

### map

map 类型是单独一类的处理方法，可以使用 reflect.ValueOf(map).Len() 拿到长度。但是，不能用下标进行遍历。遍历方式有两种。

第一种是先通过 reflect.ValueOf(map).MapKeys() 拿到所有的键，通过遍历所有的键，去一个一个找值。

MapKeys() 会返回一个值信息的切片，可以配合 Len() 拿到的长度，就可以通过下标访问每一个键。

把拿到的键传给 reflect.ValueOf(map).MapIndex(键)，就可以拿到键对应的值的值信息。

这样 reflect.ValueOf(map).MapIndex(键).Interface() 就可以拿到具体的值了。

第二种是通过 reflect.ValueOf(map).MapRange() 拿到一个有点像单链表的可以遍历的结构。

然后，一个一个处理就好了。通过 t4MapRange.Next() 判断元素有没有遍历到头，有的话会返回 true。

然后，用 t4MapRange.Key().Interface() 可以拿到键，用 t4MapRange.Value().Interface() 可以拿到值。

这就处理完一个键值对了，然后，再调用 t4MapRange.Next() 判断元素有没有遍历到头，有的话继续。

代码示例：**{demo-golang}/demo/reflectkn/map_test.c**

### 指针类型

指针类型会麻烦一点，有两种情况，第一种是一级指针，第二种是多级指针。比如，*int 类型和 **int 类型。

指针类型的 reflect.Type() 和 reflect.Value() 都可以使用 Elem()，分别拿到各自解引用后的结果。

代码示例：**{demo-golang}/demo/reflectkn/int_test.c**

#### 一级指针

比如，*int 类型。

reflect.TypeOf(*int).Kind() 拿到的类型是 reflect.Pointer。

reflect.TypeOf(*int).Elem().Kind() 拿到的类型才是 reflect.Int。

reflect.ValueOf(*int).Elem().Interface() 直接就能拿到 interface{} 类型的值。

因为，Elem() 已经相当于解引用了，所以，这里直接就能拿到具体的值。

#### 多级指针

比如，**int 类型。需要连续进行 Elem()，直到拿到具体的类型。

reflect.TypeOf(**int).Elem().Kind() 拿到的类型是 reflect.Pointer。

也就是说，对 **int 的类型信息进行 Elem()，拿到的是 *int 的类型，还是个 reflect.Pointer。

要再来一次 Elem()，也就是 reflect.TypeOf(**int).Elem().Elem().Kind() 拿到的类型才是 reflect.Int。

多级指针的这个过程可以用循环去处理。最后和一级指针一样，通过 Interface() 就能拿到 interface{} 类型的值。

### 实例和指针的关系

见图：**reflect.drawio.html 2-2**

### 方法

方法本身是可以匿名进行调用的，就像下面这样。

```
testFunc := func() {
    fmt.Println("testFunc")
}
testFunc()
```

这种方式处理有入参和出参的方法是很麻烦的，在调用的时候，必须知道变量里存的到底是什么类型的方法。如果方法的类型在不一样，那就基本没办法处理了。

这个时候，就需要用到反射了。通过反射，可以知道一个方法的入参和出参分别有几个，分别是什么类型的。

知道了这些，就可以根据方法的入参，构造对应的参数传入，也可以根据方法的出参，构造对应的参数来接收。

比如，糙一点的用法。可以通过 reflect.TypeOf(func).NumIn() 知道方法有几个入参。

然后，构造一个入参切片 `inputSlice := make([]reflect.Value, num)`，把所有的参数用 reflect.ValueOf(参数) 转换成反射的值类型，放进去。

再构造一个出参切片 `outputSlice := make([]reflect.Value, 0)`。然后，就可以调用方法了 `outputSlice = reflect.ValueOf(func).Call(inputSlice)`。

代码示例：

- **{demo-golang}/demo/reflectkn/func.c**
- **{demo-golang}/demo/reflectkn/func_test.c**

### 结构体

关于结构体的反射有两个部分，一部分是和结构体实例有关系的，一部分是和结构体本身有关系的（属性和方法）。

代码示例：

- **{demo-golang}/demo/reflectkn/struct.c**
- **{demo-golang}/demo/reflectkn/struct_test.c**

#### 结构体实例

结构体实例有两种情况，第一种直接就是结构体、第二种是结构体指针。

如果直接就是结构体的话，reflect.TypeOf(struct).Kind() 得到的是 reflect.Struct 类型。

如果直接就是结构体指针的话，处理方式和上面指针类型那里是一样的。

#### 结构体属性

通过 reflect.TypeOf() 和 reflect.ValueOf() 先拿到结构体的类型信息和值信息。

然后，通过 reflect.TypeOf(struct).NumField() 可以拿到结构体属性的数量。

知道数量之后，就可以通过 reflect.TypeOf(struct).Field(i) 和 reflect.ValueOf(struct).Field(i)，分别获取每一个属性的类型信息和值信息。

需要注意的是，私有属性是拿不到值信息的，只能拿到类型信息。通过 reflect.TypeOf(struct).Field(i).IsExported() 可以判断是不是私有属性。

reflect.TypeOf(struct).Field(i).Name 可以获取属性的名称，reflect.Zero(reflect.TypeOf(struct).Field(i).Type).Interface() 可以用类型信息构造零值。

反射还可以用来修改属性的值。很多框架都需要用到，通过属性名称找到并修改属性值，这样的操作。

reflect.TypeOf(struct).FieldByName(属性名称) 这样就可以通过属性名称，找到对应的属性的值信息。注意，这里通过结构体的类型信息，找到的是属性的值信息。

然后，用 reflect.TypeOf(struct).FieldByName(属性名称).CanSet() 可以用来判断属性能不能进行赋值操作。

如果可以的话，可以通过 reflect.TypeOf(struct).FieldByName(属性名称).Set(reflect.ValueOf(属性值)) 把值放进去。

其中 reflect.ValueOf(属性值) 这个部分，就是把具体的数据类型转换成反射使用的数据类型。

#### 结构体属性的 Tag

通过 reflect.TypeOf(struct).Field(i).Tag 就可以拿到结构体某个属性上的 Tag，这里拿到的是 reflect.StructTag。

然后，在通过 reflect.StructTag 的 Get() 方法，就可以拿到整个 Tag 里面某个具体的 Tag 的内容了。

```
type User struct {
	Id  int  `orm:"column_name=user_id"`
}
```

比如，对于 User 的 Id 属性来说，Field(i).Tag.Get(orm) 就会拿到字符串 "column_name=user_id"。

#### 结构体方法

结构体方法的处理方式，是结构体属性的处理方式和方法的处理方式相结合。

先通过 reflect.TypeOf(struct).NumMethod() 知道结构体有几个方法。

然后，就可以通过下标进行遍历了。reflect.TypeOf(struct).Method(i) 这样就能拿到方法的类型信息。

Method(i).NumIn() 和 Method(i).NumOut() 分别可以拿到方法的入参数量和出参数量。

Method(i).Type.In(i) 和 Method(i).Type.Out(i) 分别可以拿到方法每个入参的参数类型和每个出参的参数类型。

调用结构体方法和调用方法那里有点区别 `outSlice = Method(i).Func.Call(inputSlice)`。

## 参考（reference）

- {极客时间}/[Go 实战训练营](https://u.geekbang.org/subject/go2nd)
  - 反射部分
