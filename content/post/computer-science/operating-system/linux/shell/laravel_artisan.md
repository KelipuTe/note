---
draft: false
date: 2020-10-19 08:00:00 +0800
lastmod: 2020-10-19 08:00:00 +0800
title: "【Linux Shell 脚本】laravel 的 artisan 进程"
summary: "【Linux Shell 脚本】laravel 的 artisan 进程"
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
## 正文

### 保证只有一个 schedule 再跑

```shell
#!/bin/bash

# 进入项目目录
cd {project path}
# 查看是否有名字是 schedule 的进程，有说明上一个还没跑完
ps auwx | grep 'schedule' | grep -v grep
# 如果没有查到结果，说明没有在跑，就并执行 schedule
if [ "$?" != "0" ]; then
php artisan schedule run > /dev/null 2>&1 &
fi
```

### 重启所有的 queue

```shell
#!/bin/bash

# 进入项目目录
cd {project path}
# 平滑关闭所有的 queue
php artisan queue:restart
# 停 2 秒，这里不是必须的
sleep 2
# 需要处理的队列名
queue_name_list=("queue1" "queue2" "queue3")
# 循环依次把队列拉起来
for queue_name in ${queue_name_list[@]};
do
    echo "start $queue_name ..."
    nohup php artisan queue:work database --queue=$queue_name >> /tmp/$queue_name.log &
    echo "start $queue_name done"
done

exit;
```

### 确保 queue 存活

```shell
#!/bin/bash

# 进入项目目录
cd {project path}
# 停 30 秒，这里是为了和整点的 shell 错开
sleep 30
# 需要处理的队列名
queue_name_list=("queue1" "queue2" "queue3")
# 循环依次检查
for queue_name in ${queue_name_list[@]};
do
# 查看是否有名字是 $queue_name 的进程，有说明进程还活着
ps auwx | grep $queue_name | grep -v grep
# 如果没有查到结果，说明进程挂掉了，重新拉起来
if [ "$?" != "0" ]; then
nohup /usr/local/php/bin/php artisan queue:work database --queue=$queue_name >> /tmp/$queue_name.log &
fi
done

exit;
```
