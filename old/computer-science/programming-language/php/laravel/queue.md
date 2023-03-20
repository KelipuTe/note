## 队列相关

- 2021-11-08
- linux
- laravel8

### 启动laravel队列

在系统后台不挂断地启动laravel队列并输出队列进程的输出到文件中。

```shell
nohub {.../php} artisan queue:work {queue_engine} --queue={queue_name} >> {log_file} &
```

### 批量重启laravel队列，方式1

这个脚本的思路是，使用laravel提供的`queue:restart`命令平滑终止所有的队列进程，然后根据定义的队列名重新开启队列进程。

中间间隔1秒，没啥特殊的作用，只是为了隔开一1秒。

```bash
#!/bin/bash 

cd {.../project/}

{.../php} artisan queue:restart
echo "stop all queue porcess"

sleep 1

reload_queue_name=("queue1" "queue2")
for queue_name in ${reload_queue_name[@]};
do
    nohup {.../php} artisan queue:work {queue_engine} --queue=$queue_name >> /tmp/queue_output/$queue_name.log &
    echo "start $queue_name queue porcess"
done

exit;
```

### 批量重启laravel队列，方式2

这个脚本的思路是，根据定义的队列名找到队列进程，然后杀掉进程，最后重新开启队列进程。中间间隔1秒，没啥特殊的作用，只是为了隔开一1秒。

```bash
#!/bin/bash

cd {.../project/}

sleep 1

reload_queue_name=("queue1" "queue2")
for queue_name in ${reload_queue_name[@]};
do
    ps -ef | grep $queue_name | grep -v grep | awk '{print "kill "$2}' | sh
    echo "kill $queue_name queue porcess"
    nohup {.../php} artisan queue:work {queue_engine} --queue=$queue_name >> /tmp/queue_output/$queue_name.log &
    echo "start $queue_name queue porcess"
done

exit;
```