
### linux进程内存布局

- 内核空间（kernel space）
- 栈（stack）
- 动态库的映射
- 堆（heap）
- 读写数据区，主要是程序数据，`.data`、`.bss` 等
- 只读数据区，主要是程序指令，`.text`、`.init`、`.rodata` 等
- 保留区

### 动态库

动态库又称为共享库，按elf格式存储，存储的内容主要是程序指令和程序数据。

动态库的使用方式：引入头文件（声明和定义）；加载这个库文件（静态库，动态库，共享库）。

可以通过ldd命令列出一个程序所需要得动态链接库。这里用之前的hello文件：`ldd hello`。

```
linux-vdso.so.1 =>  (0x00007ffeb1fcb000)
libc.so.6 => /lib64/libc.so.6 (0x00007f3a77af6000)
/lib64/ld-linux-x86-64.so.2 (0x00007f3a77ec4000)
```
