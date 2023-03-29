---
draft: false
date: 2022-06-12 08:00:00 +0800
lastmod: 2022-06-12 08:00:00 +0800
title: "在 Go 项目中使用 Protocol Buffers"
summary: "在 Go 项目中使用 Protocol Buffers"
toc: true

categories:
- golang

tags:
- computer-science(计算机科学)
- programming-language(编程语言)
- golang
- protocol-buffers
---

### 在 Windows 11 中使用

#### 准备工作

- 安装编译器：protoc
- 安装 go 插件：protoc-gen-go

**安装编译器**

[https://developers.google.com/protocol-buffers/docs/downloads](https://developers.google.com/protocol-buffers/docs/downloads)

这个页面有怎么下载编译器的导航，按需求找对应的版本就行。

64 位的 windows 下载的应该是 **protoc-xxx-win64.zip** 这样命名的压缩文件，中间的 xxx 是版本号，解压出来是个安装程序。

安装完成之后，可以通过 `protoc --version` 命令，输出版本号验证一下。

```cmd
> protoc --version
libprotoc 3.21.1
```

**安装 go 插件**

- [https://developers.google.com/protocol-buffers/docs/gotutorial](https://developers.google.com/protocol-buffers/docs/gotutorial)
- [https://developers.google.com/protocol-buffers/docs/reference/go-generated](https://developers.google.com/protocol-buffers/docs/reference/go-generated)

这两个页面都有说明怎么安装 go 插件，执行 go install 命令就行。

`go install google.golang.org/protobuf/cmd/protoc-gen-go@latest`

#### 使用

- [https://developers.google.com/protocol-buffers/docs/gotutorial](https://developers.google.com/protocol-buffers/docs/gotutorial)
- [https://developers.google.com/protocol-buffers/docs/reference/go-generated](https://developers.google.com/protocol-buffers/docs/reference/go-generated)

这两个页面都有说明怎么使用，主要就是先编写 .proto 文件，然后执行 protoc 命令。

比如，在 `.\api\v1\test\test.proto` 文件中添加如下代码。

```protobuf
syntax = "proto3";
package test;

import "google/api/annotations.proto";

option go_package = "/v1/test;v1test";

service Test {
    rpc TestGet (TestGetRequest) returns (TestGetReply) {
        option (google.api.http) = {
            get: "/api/v1/test"
        };
    }
}

message TestGetRequest {
    string key = 1;
}

message TestGetReply {
    string key = 1;
    string val = 2;
}
```

然后，再命令行执行： `protoc --proto_path=.\api --go_out=.\api .\api\v1\test\test.proto`。

这个时候不出意外会报错：`Import "google/api/annotations.proto" was not found or had errors.`。 这是因为代码里的 `import "google/api/annotations.proto";` 这一行，引入了外部文件 annotations.proto。

annotations.proto 这个文件可以从 googleapis 这个项目里找到。[https://github.com/googleapis/googleapis](https://github.com/googleapis/googleapis)

文件路径是 googleapis/google/api/annotations.proto。需要注意，annotations.proto 这个文件又引入了 http.proto。http.proto 文件的路径是 googleapis/google/api/http.proto。

把这两个文件下载下来，放到 `.\api\google\api` 目录下。 现在的目录结构变成这样：

- .\api\google\api\annotations.proto
- .\api\google\api\http.proto
- .\api\v1\test\test.proto

这个时候执行 `protoc --proto_path=.\api --go_out=.\api .\api\v1\test\test.proto`。就会在 `.\api\v1\test` 目录下面生成 **test.pb.go**。test.pb.go 里面有根据 test.proto 文件中定义的实体生成的对应的 go 的结构体代码。

`--go_out=.\api` 表示文件生成到 `.\api` 目录下面。代码中的 `option go_package = "/v1/test;v1test";`，表示文件会生成到 `--go_out` 指定的目录下面的 `\v1\test` 目录。后面的 v1test 表示生成的 go 文件的包名是 v1test。

到这里为止，并没有使用下面这段代码。

```protobuf
service Test {
    rpc TestGet (TestGetRequest) returns (TestGetReply) {
        option (google.api.http) = {
            get: "/api/v1/test"
        };
    }
}
```

这段代码可以用于生成 grpc 服务的代码和 http 服务的代码。在使用 protoc 时，加上 `--go-grpc_out=.\api` 选项，可以生成 grpc 服务的代码，文件名称是 test_grpc.pb.go。在使用 protoc 时，`--go-http_out=.\api` 选项，可以生成 http 服务的代码，文件名称是 test_http.pb.go。

生成的 http 服务的代码会导入 kratos 的包是因为，引用了 proto-gen-go-http，并且依赖 google api 定义了接口。这个是 kratos 提供的 proto-gen-go-http 生成的。

#### reference（参考）

- 概述：[https://developers.google.com/protocol-buffers/docs/overview](https://developers.google.com/protocol-buffers/docs/overview)
- protobuf-go：[https://github.com/protocolbuffers/protobuf-go](https://github.com/protocolbuffers/protobuf-go)
- go 基础使用：[https://developers.google.com/protocol-buffers/docs/gotutorial](https://developers.google.com/protocol-buffers/docs/gotutorial)
- 编译器下载：[https://developers.google.com/protocol-buffers/docs/downloads](https://developers.google.com/protocol-buffers/docs/downloads)
- 编译器下载：[https://github.com/protocolbuffers/protobuf/releases](https://github.com/protocolbuffers/protobuf/releases)
- go 生成代码：[https://developers.google.com/protocol-buffers/docs/reference/go-generated](https://developers.google.com/protocol-buffers/docs/reference/go-generated)


- [proto编译引用外部包问题](https://www.cnblogs.com/yisany/p/14875488.html)
- [解决：Import googleapiannotations.proto was not found or had errors](https://blog.csdn.net/weixin_44829930/article/details/124079183)
- googleapis/googleapis：[https://github.com/googleapis/googleapis](https://github.com/googleapis/googleapis)
- grpc-ecosystem/grpc-gateway：[https://github.com/grpc-ecosystem/grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)


- go-kratos/kratos：[https://github.com/go-kratos/kratos](https://github.com/go-kratos/kratos)
- 另外，感谢极客时间 Go 进阶训练营的助教包子老师的解答。

### 在 Ubuntu 20.04 中使用

#### 准备工作

- 安装编译器：protoc
- 安装 go 插件：protoc-gen-go

**安装编译器**

linux 需要通过源码安装，下载的应该是 **Source code (tar.gz)** 源码压缩文件。解压之后，编译安装。

```shell
# 安装需要的工具包
> apt-get install autoconf automake libtool
# 解压
> tar xvf protobuf-21.1.tar.gz
> cd protobuf-21.1/
# 编译安装
> ./autogen.sh
> ./configure
> make
> make install
# 输出版本号验证一下
> protoc --version
```

不成功的，可以试试再来一遍，说不定就成功了。不知道为什么，挺玄学的。

#### reference（参考）

- [Ubuntu 20.04 配置 go 使用protobuf](https://blog.csdn.net/weixin_43266367/article/details/121614008)
- [ubuntu20.04安装protobuf](https://blog.csdn.net/m0_60827485/article/details/125002366)
