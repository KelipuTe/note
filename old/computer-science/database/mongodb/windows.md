## windows环境安装和配置mongodb

- 2021-10-21
- windows10
- mongodb5.0.3

### 下载

去[mongodb官方网站(https://www.mongodb.com/try/download/community)](https://www.mongodb.com/try/download/community)下载一个windows环境的解压版的mongodb。
这里下的是windows环境的解压版的mongodb5.0.3(mongodb-windows-x86_64-5.0.3.zip)。

### 解压版安装

下载完成后解压到自定义目录。进入mongodb目录。

新建名为data的目录，用于存放数据。

新建名为mongodb.conf的配置文件，简单配置一下。

```conf
#端口号，默认为27017
port=27017
#数据库存放路径
dbpath=...\data
#日志存放路径
logpath=...\data\mongodb.log
```

### 命令行启动

以管理员模式打开cmd，进入mongodb目录下的bin目录。

执行命令`mongod --config "E:\mongodb-5.0.3\mongodb.conf"`，使用配置文件的配置启动mongodb服务。执行后服务其实已经起来了，但是cmd可能没有明显的反应。

以管理员模式打开另一个cmd，进入mongodb目录下的bin目录。

执行命令`mongo`，登入mongodb。成功登入可以看到下面的输出并进入命令行操作界面。

```shell
MongoDB shell version v5.0.3
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("2d7d9bbd-8a80-4119-a4dc-85b789f19e51") }
MongoDB server version: 5.0.3
================
Warning: the "mongo" shell has been superseded by "mongosh",
which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
an upcoming release.
We recommend you begin using "mongosh".
For installation instructions, see
https://docs.mongodb.com/mongodb-shell/install/
================
---
The server generated these startup warnings when booting:
        2021-10-21T14:38:02.917+08:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
        2021-10-21T14:38:02.917+08:00: This server is bound to localhost. Remote systems will be unable to connect to this server. Start the server with --bind_ip <address> to specify which IP addresses it should serve responses from, or with --bind_ip_all to bind to all interfaces. If this behavior is desired, start the server with --bind_ip 127.0.0.1 to disable this warning
---
---
        Enable MongoDB's free cloud-based monitoring service, which will then receive and display
        metrics about your deployment (disk utilization, CPU, operation statistics, etc).

        The monitoring data will be available on a MongoDB website with a unique URL accessible to you
        and anyone you share the URL with. MongoDB may use this information to make product
        improvements and to suggest MongoDB products and deployment options to you.

        To enable free monitoring, run the following command: db.enableFreeMonitoring()
        To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
>
```

> connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb

这行就是mongodb服务的连接地址，用连接工具(如MongoDBCompass)连接的时候就用这个，这里没有设置密码，可以直接连。
