## 一些异常和解决方案

- 2021-11-17
- windows10
- mysql-8.0.27

### MySQL8.0

navicat连接mysql报错：

```
Authentication plugin 'caching_sha2_password' cannot be loaded
```

#### 解决方案1

适用场景：第一次构建容器/安装、已安装完成后新增用户。

配置mysql.cnf配置默认身份验证插件：

```
[mysqld]
default_authentication_plugin = mysql_native_password
```

#### 解决方案2

适用场景：mysql已成功安装完成后。

查看身份验证类型：

```
mysql> use mysql;
Database changed

mysql> SELECT Host, User, plugin from user;
+-----------+------------------+-----------------------+
| Host      | User             | plugin                |
+-----------+------------------+-----------------------+
| %         | root             | caching_sha2_password |
| localhost | mysql.infoschema | caching_sha2_password |
| localhost | mysql.session    | caching_sha2_password |
| localhost | mysql.sys        | caching_sha2_password |
| localhost | root             | caching_sha2_password |
+-----------+------------------+-----------------------+
5 rows in set (0.00 sec)
```

发现root用户的验证器插件为caching_sha2_password。

修改身份验证类型(同时会修改密码)。

```
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.01 sec)
```

让设置生效。

```
mysql> FLUSH PRIVILEGES;
```

验证是否生效。

```
mysql> SELECT Host, User, plugin from user;
+-----------+------------------+-----------------------+
| Host      | User             | plugin                |
+-----------+------------------+-----------------------+
| %         | root             | mysql_native_password |
| localhost | mysql.infoschema | caching_sha2_password |
| localhost | mysql.session    | caching_sha2_password |
| localhost | mysql.sys        | caching_sha2_password |
| localhost | root             | mysql_native_password |
+-----------+------------------+-----------------------+
5 rows in set (0.00 sec)
```
