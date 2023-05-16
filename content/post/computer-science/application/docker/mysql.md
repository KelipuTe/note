---
draft: false
create_date: 2022-05-18 08:00:00 +0800
date: 2022-10-17 08:00:00 +0800
title: "使用 Docker 启动 MySQL"
summary: "拉取容器；启动容器；异常处理；"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- application(应用)
- docker
- database(数据库)
- mysql
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- Docker v20.10.17

## 正文

### 拉取镜像

先去 DockerHub 找想要的容器。直接搜索 mysql 就能找到，[链接](https://hub.docker.com/_/mysql) 在这。

如果不讲究，那直接用最上面提供的命令就可以了。如果想要某个版本，就点到 Tags 标签页里面去，找到想要的版本，然后用相应的命令。

这里用的是 mysql8，输入命令后等着下载就好了。

```
> docker pull mysql:8.0

8.0: Pulling from library/mysql
...
Digest: sha256:147572c972192417add6f1cf65ea33edfd44086e461a3381601b53e1662f5d15
Status: Downloaded newer image for mysql:8.0
docker.io/library/mysql:8.0
```

拉好之后，可以在 "docker images" 命令输出的列表里看到。

```
> docker images

REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
mysql        8.0       40b83de8fb1a   4 days ago   535MB
```

### 启动容器

通过 "docker run" 命令启动容器。

```
> docker run -itd --name mysql8-dev -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:8.0

45804f67e3bf4b79735c308a8073131525c5fbc6f714e8fca5efb4f628127532
```

- "-itd" 参数，在后台运行容器，并且打印容器的 ID。
- "--name mysql8-dev"，设置容器的名字为 mysql8-dev。
- "-p 3306:3306"，将本机的 3306 端口和容器的 3306 端口进行映射。
- "-e MYSQL_ROOT_PASSWORD=123456"，设置 mysql 密码为 123456，用户名默认为 root。
- "mysql:8.0"，使用 mysql:8.0 镜像启动容器。

启动好之后，可以通过 "docker ps" 命令，查看一下容器状态。

### 进入容器

使用 "docker exec" 命令进入容器。

```
> docker exec -it mysql8-dev /bin/bash

bash-4.4# 
```

mysql8-dev 就是上面设置的容器的名字，也可以用容器的 ID。

使用默认的用户名和上面设置的密码访问 mysql。

```
> bash-4.4# mysql -uroot -p123456

mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.31 MySQL Community Server - GPL

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
