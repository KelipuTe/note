---
draft: false
date: 2020-10-19 08:00:00 +0800
lastmod: 2020-10-19 08:00:00 +0800
title: "【Linux Shell 脚本】压缩日志"
summary: "【Linux Shell 脚本】压缩日志"
toc: true

categories:
- shell

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- shell
---
## 正文

#### 每天压缩前一天的日志

```shell
#!/bin/bash

# 获取前一天的 yyyy-mm-dd 格式的日期
dir_date=$(date -d yesterday +%Y-%m-%d)
# 进入日志目录对应日期的目录
cd /tmp/logs/$dir_date/
# 查找所有以 .log 为后缀名的文件，并压缩
find ./ -name '*.log' | xargs -i gzip {}

exit;
```
