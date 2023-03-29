## grep

- 2021-10-19

grep命令可用于查找文件里符合条件的字符串。

通常和管道（`|`）一起用。

### 参数

- `-A`：除了显示符合范本样式的那一列之外，并显示该行之后的内容。
- `-B`：除了显示符合样式的那一行之外，并显示该行之前的内容。
- `-C`：除了显示符合样式的那一行之外，并显示该行之前后的内容。 

```shell
#在fileName文件中查找string，显示string所在行及后5行的数据
grep -A 5 string fileName
#在fileName文件中查找string，显示string所在行及前5行的数据
grep -B 5 string fileName
#在fileName文件中查找string，显示string所在行及前后5行的数据
grep -C 5 string fileName
```