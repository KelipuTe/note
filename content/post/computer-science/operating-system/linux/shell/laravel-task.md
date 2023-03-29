---
draft: false
date: 2020-10-19 08:00:00 +0800
lastmod: 2020-10-19 08:00:00 +0800
title: "Laravel 定时任务 shell 脚本"
summary: "Laravel 定时任务 shell 脚本"
toc: true

categories:
- shell

tags:
- computer-science(计算机科学)
- operating-system(操作系统)
- linux
- shell
- laravel
---

这段脚本用于保证 laravel 的 schedule 定时任务，同时只有一个在跑。

```shell
#!/bin/bash

# 查看是否有名字是 schedule 进程，有说明上一个还没跑完
ps auwx | grep 'schedule' | grep -v grep
# 如果没有查到结果，说明没有在跑，就进入项目目录并执行 schedule
if [ "$?" != "0" ]; then
cd /project/
php artisan schedule run > /dev/null 2>&1 &
fi
```
