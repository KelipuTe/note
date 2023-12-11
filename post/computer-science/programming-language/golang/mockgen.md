---
draft: false
date: 2023-04-24 08:00:00 +0800
create_date: 2022-12-10 08:00:00 +0800
title: "使用 gomock 工具进行单元测试"
summary: "gomock 工具的安装和使用；"
toc: true

categories:
- golang

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- golang
- test
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- go version go1.19 windows/amd64
- github.com/golang/mock v1.6.0

## 正文

gomock 是测试框架；mockgen 是代码生成工具，可以根据接口的规范自动生成代码。

### 安装 mockgen

(https://github.com/golang/mock)，github 上的 README 就是文档。

go 版本大于 1.16 的，使用 `go install github.com/golang/mock/mockgen@v1.6.0` 命令安装。装完就可以用了，直接命令行窗口输入 "mockgen" 命令，会得到下面的输出。

```
> mockgen

mockgen has two modes of operation: source and reflect.

Source mode generates mock interfaces from a source file.
It is enabled by using the -source flag. Other flags that
may be useful in this mode are -imports and -aux_files.
Example:
        mockgen -source=foo.go [other options]

Reflect mode generates mock interfaces by building a program
that uses reflection to understand interfaces. It is enabled
by passing two non-flag arguments: an import path, and a
comma-separated list of symbols.
Example:
        mockgen database/sql/driver Conn,Driver

  -aux_files string
        (source mode) Comma-separated pkg=path pairs of auxiliary Go source files.
  -build_flags string
        (reflect mode) Additional flags for go build.
  -copyright_file string
        Copyright file used to add copyright header
  -debug_parser
        Print out parser results only.
  -destination string
        Output file; defaults to stdout.
  -exec_only string
        (reflect mode) If set, execute this reflection program.
  -imports string
        (source mode) Comma-separated name=path pairs of explicit imports to use.
  -mock_names string
        Comma-separated interfaceName=mockName pairs of explicit mock names to use. Mock names default to 'Mock'+ interfaceName suffix.
  -package string
        Package of the generated code; defaults to the package of the input with a 'mock_' prefix.
  -prog_only
        (reflect mode) Only generate the reflection program; write it to stdout and exit.
  -self_package string
        The full package import path for the generated code. The purpose of this flag is to prevent import cycles in the generated code by trying to include its own package. This can happen if the mock's package is set to one of its
 inputs (usually the main one) and the output is stdio so mockgen cannot detect the final output package. Setting this flag will then tell mockgen which import to exclude.
  -source string
        (source mode) Input Go source file; enables source mode.
  -version
        Print version.
  -write_package_comment
        Writes package documentation comment (godoc) if true. (default true)
2022/12/12 10:00:27 Expected exactly two arguments
```

### 生成 mock 文件

参数很多但是一般就用两个："-package" 指定生成的 go 文件的包名；"-destination" 指定生成的 go 文件的位置和文件名。
比如，给 go-redis 的 Cmdable 接口生成 mock，就可以在项目的根目录输入下面的命令。

`mockgen -package=mock -destination="mock/redis_cmdable.mock.go" github.com/redis/go-redis/v9 Cmdable`。

这个命令就表示给 go-redis 的 Cmdable 接口生成 mock。生成的 go 文件的包名为 "package mock"。生成的文件在 "./mock/" 目录，文件名为 redis_cmdable.mock.go。
有的时候，运行命令之后，文件不是立刻生成的，需要等一会。另外，生成的过程会创建临时目录和临时文件，它们会被自动删掉的，不用管。

如果上面的命令没有生成文件，就先试试不加参数的能不能执行：`mockgen github.com/go-redis/redis/v9 Cmdable`。这个命令会使用默认配置生成 mock 文件。

如果碰到下面这种报错：

```
prog.go:12:2: no required module provides package github.com/golang/mock/mockgen/model; to add it:
go get github.com/golang/mock/mockgen/model
prog.go:12:2: no required module provides package github.com/golang/mock/mockgen/model; to add it:
go get github.com/golang/mock/mockgen/model
prog.go:14:2: no required module provides package github.com/go-redis/redis/v9: go.mod file not found in current directory or any parent directory; see 'go help modules'
prog.go:12:2: no required module provides package github.com/golang/mock/mockgen/model: go.mod file not found in current directory or any parent directory; see 'go help modules'
2022/12/10 16:40:01 Loading input failed: exit status 1
```

报错信息里面已经提示了，就执行一下 `go get github.com/golang/mock/mockgen/model` 命令就行了。

### 在单元测试里使用

```
func TestXXX(t *testing.T) {
    // 开头这里 NewController 和 Finish 是固定要调用一下的
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()
    
    // 这个方法是 mockgen 生成的，在 mock/redis_cmdable.mock.go 里面
    cmdable := mock.NewMockCmdable(ctrl)
    
    // 定义 redis 语句执行的结果
    cmd := redis.NewCmd(context.Background(), nil)
    cmd.SetVal("OK")
    
    // 把上面定义的执行结果，指定给一个语句
    cmdable.EXPECT().Eval(gomock.Any(), {redis 语句}, 语句需要的参数 1, 语句需要的参数 2, ..., gomock.Any()).Return(cmd)
}
```

这么就定义好了，调用 redis 执行指定的 redis 语句的 mock 数据。注意，测试的时候调用的语句和语句需要的参数必须是一样的。而且，上面这种定义方式这里只能调用一次。

```
cmdable := mock.NewMockCmdable(p7ctrl)

cmd := redis.NewCmd(context.Background(), nil)
cmd.SetErr(context.DeadlineExceeded)
cmdable.EXPECT().Eval(gomock.Any(), {redis 语句 1}, 语句需要的参数 1, 语句需要的参数 2, ..., gomock.Any()).Times(2).Return(cmd)

cmd2 := redis.NewCmd(context.Background(), nil)
cmd2.SetVal("OK")
cmdable.EXPECT().Eval(gomock.Any(), {redis 语句 2}, 语句需要的参数 1, 语句需要的参数 2, ..., gomock.Any()).Return(cmd2)
```

可以用 Times() 方法指定语句的调用次数，如果无限次的话就用 AnyTimes()。如果需要设置多种语句执行的结果，那就像上面那样，挨个定义，挨个设置就行。

用 Times() 的时候，不知道需不需要控制调用次数。这里用的时候发现，如果设置的调用次数没用完测试就结束了，那单元测试会报错。调整一下调用次数和被测试的程序保持一致就行。

```
controller.go:269: missing call(s) to 
中间一大堆，应该就是调用次数没用完那个 redis 语句
controller.go:269: aborting test due to missing call(s)
```

如果设置了，但是没有调用的话，就会报下面这样的错：

```
controller.go:269: missing call(s) to 一大堆，会具体说哪个接口没调用
controller.go:269: aborting test due to missing call(s)
```
