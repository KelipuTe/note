---
draft: false
date: 2022-06-07 08:00:00 +0800
lastmod: 2022-06-07 08:00:00 +0800
title: "在 Linux 中安装 MySQL"
summary: "安装 MySQL 8；"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- vmware
- vmware-workstation
- linux
- ubuntu
- mysql
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> VMware Workstation Pro 16<br/>
> Ubuntu 22.04

### 前言

没有 Linux 环境的，可以装个虚拟机，然后在里面玩玩： [在 VMware 虚拟机中安装 Linux](/post/computer-science/application/vmware/linux)

### 更新源

```
> apt update
Reading package lists... Done
E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
E: Unable to lock directory /var/lib/apt/lists/
W: Problem unlinking the file /var/cache/apt/pkgcache.bin - RemoveCaches (13: Permission denied)
W: Problem unlinking the file /var/cache/apt/srcpkgcache.bin - RemoveCaches (13: Permission denied)
```

`Permission denied`，非生产环境，一般直接 sudo 就行。

```
> sudo apt update
...
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
159 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

看到这个就成功了。

### 安装 MySql 8

```
> sudo apt install mysql-server
...
mysqld will log errors to /var/log/mysql/error.log
mysqld is running as pid 3171
Created symlink /etc/systemd/system/multi-user.target.wants/mysql.service → /lib
/systemd/system/mysql.service.
Setting up mysql-server (8.0.29-0ubuntu0.22.04.2) ...
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3) ...
```

看到这个就成功了。安装完成，会直接启动。

可以通过查看 MySQL 版本确认一下。

```
> mysql -V
mysql  Ver 8.0.29-0ubuntu0.22.04.2 for Linux on x86_64 ((Ubuntu))
```

也可以通过 `ps` 命令确认一下。

```
> ps aux | grep mysql
mysql       3357  0.2  9.8 1780000 391948 ?      Ssl  22:46   0:04 /usr/sbin/mysqld
```

### 修改 MySQL 密码

安装 MySQL 的时候会生成一个默认密码。

```
> sudo cat /etc/mysql/debian.cnf

# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = debian-sys-maint
password = flP8kWluyyOyvs9Y
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = flP8kWluyyOyvs9Y
socket   = /var/run/mysqld/mysqld.sock
```

其中 user 和 password 就对应用户名和密码。

```
# 连接 MySQL
> mysql -udebian-sys-maint -pflP8kWluyyOyvs9Y

# 修改密码
mysql> use mysql;
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
```

### 让 MySQL 可以远程访问

#### MySQL 配置文件

没有配置远程访问前，通过 netstat 命令查看 MySQL 的端口是这样的：

```
> netstat -ano | grep 3306
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      off (0.00/0/0)
```

这里的 127.0.0.1:3306 表示 MySQL 只接收本地的连接。

打开 MySQL 配置文件：`sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf`。

```
...
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 127.0.0.1
mysqlx-bind-address     = 127.0.0.1
...
```

- 找到 bind-address，原来是 127.0.0.1，改成 0.0.0.0。
- 然后重启 MySQL 服务：`sudo service mysql restart`。

配置远程访问后，通过 netstat 命令查看 MySQL 的端口是这样的：

```
> netstat -ano | grep 3306
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      off (0.00/0/0)
```

这里的 0.0.0.0:3306 表示 MySQL 现在可以和外部建立连接。

#### MySQL 配置

登入 MySQL 实例，给 root 用户设置远程连接权限。

```
mysql> use mysql;
mysql> select user,host from user;

+------------------+-----------+
| user             | host      |
+------------------+-----------+
| debian-sys-maint | localhost |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
5 rows in set (0.00 sec)
```

没设置前 root 用户对应的 host 为 "localhost"。

```
mysql> update user set host='%' where user='root' and host='localhost';

Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select user,host from user;

+------------------+-----------+
| user             | host      |
+------------------+-----------+
| root             | %         |
| debian-sys-maint | localhost |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
+------------------+-----------+
5 rows in set (0.00 sec)
```

将 root 用户对应的 host 设置为 "%" 表示可以进行远程连接。

```
mysql> flush privileges;
```

别忘了，刷新数据库。

#### 配置虚拟机防火墙

查看防火墙规则列表。

```
> sudo iptables -L

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

设置 3306 端口外部可以访问。

```
udo iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
```

设置完会变成下面这样：

```
> sudo iptables -L

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:mysql

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

需要注意的是，用命令配置是即时生效的，重启以后规则会丢失。想要让配置永久有效，可以去修改 iptables 配置文件。命令 `service iptables save` 也可以让 iptables 设置长久有效，重启后不丢失。

#### 虚拟机端口映射

- 选中需要修改的虚拟机
- 最上面的导航栏 --> 点击 "编辑" --> 点击 "虚拟网络编辑器"

打开命令行，使用 `ifconfig` 命令查看虚拟机在子网中的 IP。这里有可能会有很多个 IP，单看这里可能无法确定。

```
> ifconfig

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.226.128  netmask 255.255.255.0  broadcast 192.168.226.255
        inet6 fe80::7dd9:3248:6626:cde9  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:5c:43:29  txqueuelen 1000  (Ethernet)
        RX packets 607  bytes 57711 (57.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 283  bytes 28521 (28.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 266  bytes 22796 (22.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 266  bytes 22796 (22.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

- 虚拟网络编辑器 --> 在最上面的列表里面找到 VMnet8（子网地址是 192.168.226.0 的那个）。从上面 ifconfig 命令的结果里面，找到在子网地址下的那个（这里就是 192.168.226.128）
- 在下面的 "VMnet 信息" 里面--> 选择 "NAT 模式" --> 点击右边的 "NAT 设置" 按钮。如果按钮不让点，就先点一下最下面的 "更改设置"。
- 端口转发 --> 点击 "添加" --> 添加一个主机端口和虚拟机端口的映射（这里把主机 3306 映射到虚拟机 3306），虚拟机 IP 地址就填上面的 192.168.226.128。

#### 测试

搞完上面的步骤，如果没问题的话，理论上就可以从主机访问虚拟机的 MySQL 服务了。连接有点慢，应该和虚拟机环境有关系。

### 参考

- [Ubuntu安装mysql8, 修改mysql密码，配置远程连接](https://blog.csdn.net/weixin_54199229/article/details/124100059)
- [虚拟机端口映射提供mysql远程服务](https://www.freesion.com/article/5782568266/)
- [如何在Ubuntu20.04上安装MySQL以及如何配置MySQL的远程连接](https://www.bilibili.com/read/cv16103674/)
- [Ubuntu 开启Mysql 3306端口远程访问](https://blog.csdn.net/viviliving/article/details/124120708)
