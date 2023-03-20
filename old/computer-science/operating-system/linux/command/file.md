## file

- 2021-12-01

file命令用于辨识文件类型。

### 安装

```shell
yum -y install file
```

### 示例

```shell
#c源码文件
> file hello.c
hello.c: C source, ASCII text, with CRLF line terminators
#hello.o是一个可重定位文件（relocatable）
> file hello.o
hello.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
#hello是一个可执行文件（executable）
> file hello
hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=a00dd2efbcbd8e23589373d89189b9785f0b7542, not stripped
```