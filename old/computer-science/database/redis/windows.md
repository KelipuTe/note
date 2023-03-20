## windows环境安装和配置redis

### 下载

去 github 下载一个 windows 环境的解压版的 redis。注意 `redis/redis`（[https://github.com/redis/redis](https://github.com/redis/redis)）不支持 windows。`tporadowski/redis`（[https://github.com/tporadowski/redis](https://github.com/tporadowski/redis)）这个支持 windows。

这里下载下来的文件是：Redis-x64-5.0.14.zip

### 配置

下载完成后解压到合适的位置。用记事本打开解压后的目录中的配置文件 `redis.windows.conf`。一般不需要修改什么配置，这里修改一下密码。

全局搜索 `AUTH <PASSWORD>`，找到下面这段。`requirepass` 后面的就是设置的密码，去掉前面的 `#` 号启用，注意有空格的话也去掉。

```
# Require clients to issue AUTH <PASSWORD> before processing any other
# commands.  This might be useful in environments in which you do not trust
# others with access to the host running redis-server.
#
# This should stay commented out for backward compatibility and because most
# people do not need auth (e.g. they run their own servers).
#
# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
#
# requirepass foobared
```

### 命令行启动

以管理员模式打开 cmd，进入 redis 目录。执行命令`redis-server redis.windows.conf`启动 redis 服务并使用配置文件。启动成功的话会看见这个。

```
[14904] 17 Nov 13:43:37.618 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
[14904] 17 Nov 13:43:37.618 # Redis version=5.0.14, bits=64, commit=a7c01ef4, modified=0, pid=14904, just started
[14904] 17 Nov 13:43:37.618 # Configuration loaded
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 5.0.14 (a7c01ef4/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 14904
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

[14904] 17 Nov 13:43:37.624 # Server initialized
[14904] 17 Nov 13:43:37.625 * Ready to accept connections
```

### 注册服务

以管理员模式打开 cmd，进入 redis 目录，常用的操作有下面 4 个。

```
#安装redis服务
redis-server --service-install redis.windows.conf
#启动redis服务
redis-server --service-start
#停止redis服务
redis-server --service-stop
#卸载redis服务
redis-server --service-uninstall
```

安装的服务，可以从 windows 的服务管理界面（`桌面-->我的电脑-鼠标右击-->菜单-管理-->选项卡-服务和应用程序-->选项卡-服务`）中找到，名字一般就是 redis。

安装好后，需要手动执行第二行命令启动服务，也可以从上面 windows 的服务管理界面启动。

### RedisDesktopManager

RedisDesktopManager 是一个 redis 数据库桌面管理系统，类似 navicate。

可以去 github 下载`uglide/RedisDesktopManager`（https://github.com/uglide/RedisDesktopManager）。

新版本的是需要付费的，0.8.8 版本以下不用付费，0.8.8 版本地址（[https://github.com/uglide/RedisDesktopManager/releases/tag/0.8.8](https://github.com/uglide/RedisDesktopManager/releases/tag/0.8.8)）。
