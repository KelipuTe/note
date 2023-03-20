## tail

- 2021-10-19

tail命令可用于查看文件的内容。

### 参数

- `-f`：循环读取。
- `-n<行数>`：显示文件的尾部n行内容。

```shell
#持续读取fileName文件尾部的数据
tail -f fileName
#读取fileName文件尾部100行
tail -n 100 fileName
```