## 使用 go module

- 2021-07-16
- go-1.15

go module是官方自带的go依赖管理库，在1.13版本正式推荐使用。

go1.13+的版本，判断module开不开启的依据是，根目录下有没有`go.mod`文件。

使用`go env -w GO111MODULE=on`命令可以设置module全局开启。

如果使用了go module，那产生的依赖包放置在`$GOPATH/pkg/mod`目录下

### 初始化

在模块根目录下使用`go mod`命令进行初始化，初始化完成后会在模块根目录下生成`go.mod`文件。

```powershell
go mod init {module_name}
```

### 导入本地包

目录结构

```
- hello/
  - hello1/
    - helloworld1.go
  - hello2/
    - helloworld2.go
- main.go
- go.mod
```

go.mod

```go
module hello

go 1.15
```

main.go

```go
package main

import (
	"go0yang4li4/hello1"
	"go0yang4li4/hello2"
)

func main() {
	hello1.FHelloWorld()
	hello2.FHelloWorld()
}
```

helloworld1.go

```go
package hello1

import "fmt"

func FHelloWorld() {
	fmt.Println("hello world1")
}
```

helloworld2.go

```go
package hello2

import "fmt"

func FHelloWorld() {
	fmt.Println("hello world2")
}
```

