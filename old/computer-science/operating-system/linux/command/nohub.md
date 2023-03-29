## nohup

- 2021-10-19

nohup用于在系统后台不挂断地运行命令。

使用nohup命令运行应用程序，可以使其作为后台守护进程运行。

```shell
nohup php artisan serve --port=8000 > nohup_laravel.log 2>&1 &
```

- `nohup ... &`：在后台不挂断的执行命令。
- `php artisan serve --port=8000`：laravel服务的启动命令。
- `> nohup_laravel.log`：将命令的标准输出重定向到nohup_laravel.log文件
- `2>&1`：将标准错误输出重定向到标准输出。
- `> nohup_laravel.log 2>&1`：合起来用，就是把所有输出都定向到文件。

上面的命令执行后会返回进程id。
