## windows环境安装和配置mysql

- 2021-11-17
- windows10
- mysql-8.0.27

### 下载

去官方网站（https://dev.mysql.com/downloads/mysql）下一个windows环境的解压版的压缩包。

本人下载下来的文件是：mysql-8.0.27-winx64。

### 安装

将文件解压到合适的位置。

- 在桌面右击我的电脑找到属性并点击。
- 在界面左侧找到高级系统设置并点击。
- 在界面下面找到环境变量并点击。
- 在界面的系统变量列表中找到path变量并点击编辑。
- 在path变量中添加mysql的bin目录。
- 本人的这个目录是`D:\Program Files\mysql-8.0.27-winx64\bin`。

在根目录创建一个data目录，用于存放数据。

在根目录创建一个my.ini配置文件，需要写入的内容如下：

```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8

[mysqld]
#设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=D:\Program Files\mysql-8.0.27-winx64
# 设置mysql数据库的数据的存放目录
datadir=D:\Program Files\mysql-8.0.27-winx64\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```

以管理员身份打开win10的命令行窗口（cmd），进入mysql目录下的bin目录，使用`mysqld install`命令安装服务。

安装成功会输出下面这样的信息。

```
> mysqld install
Service successfully installed.
```

如果没有使用管理员身份打开cmd，会出现下面这样的报错。

```
> mysqld install
Install/Remove of the Service Denied!
```

### 启动mysql

`我的电脑=>右击=>管理=>服务和应用程序=>服务`，找到mysql服务并启动。

不出意外会发生下面的报错。本人也不知道为什么，每次新装的mysql一定会这样。

```
本地计算机上的 MySQL 服务启动后停止。某些服务在未由其他服务或程序使用时将自动停止？
```

先清空mysql根目录下的data目录。然后，用命令行进入bin目录，使用`mysqld --initialize`命令执行初始化。

初始化完成后，在设备管理器的服务管理内再次启动mysql。

## 修改登录密码

在mysql根目录下data文件夹内，找到`.err`结尾文件，用记事本打开找到如下内容：

```
A temporary password is generated for root@localhost: &Ga/k>jYy2Z1
```

`&Ga/k>jYy2Z1` 就是mysql在安装时，随机生成的root账号的密码。

用命令行登录mysql服务： `mysql -u root -p`。

可以用命令`mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';`修改密码为new_password。

也可以用命令`mysql>set password for root@localhost = password('new_password');`修改密码为new_password。