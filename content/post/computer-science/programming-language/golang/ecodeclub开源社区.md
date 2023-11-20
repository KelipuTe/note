### 提交代码前的准备工作

make setup，初始化各个工具

跑不起来的话，打开 script/setup.sh，把里面几个安装工具的命令跑一下，一般是下面这两个

go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.52.2
go install golang.org/x/tools/cmd/goimports@latest

make check，执行 go mod tidy，代码格式化
make ut，运行单元测试
make lint，运行代码检查工具
