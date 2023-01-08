---
draft: false
date: 2022-05-18 08:00:00 +0800
lastmod: 2022-10-17 08:00:00 +0800
title: "使用 Docker 部属 MySQL 8.0"
summary: "使用 Docker 部属 MySQL 8.0"
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

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> Docker v20.10.17

### 拉取镜像

访问 [Docker MySQL 镜像库](https://hub.docker.com/_/mysql/tags) 

拉取 MySQL 8.0 镜像。

```shell
> docker pull mysql:8.0

8.0: Pulling from library/mysql
...
Digest: sha256:147572c972192417add6f1cf65ea33edfd44086e461a3381601b53e1662f5d15
Status: Downloaded newer image for mysql:8.0
docker.io/library/mysql:8.0
```

拉好之后，可以在 image 列表看到。

```shell
> docker images

REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
mysql        8.0       40b83de8fb1a   4 days ago   535MB
```

### 启动容器

设置好参数，然后后台启动容器。

```shell
> docker run -itd --name mysql-8-dev -p 127.0.0.1:13306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:8.0

45804f67e3bf4b79735c308a8073131525c5fbc6f714e8fca5efb4f628127532
```

- `docker run -itd`：在后台运行容器，并且打印容器 id。
- `--name mysql-8-dev`：设置容器的名字为 mysql-8-dev。
- `-p 127.0.0.1:13306:3306`：将本机的 127.0.0.1:13306 端口和容器的 3306 端口进行映射。
- `-e MYSQL_ROOT_PASSWORD=123456`：设置 mysql 密码为 123456，用户名默认为 root。
- `mysql:8.0`：使用 mysql:8.0 镜像运行容器。

启动好容器之后，可以通过 `docker ps` 命令查看容器状态。
```shell
> docker ps

CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                                  NAMES
45804f67e3bf   mysql:8.0   "docker-entrypoint.s…"   15 seconds ago   Up 14 seconds   33060/tcp, 127.0.0.1:13306->3306/tcp   mysql-8-dev
```

### 进入容器

使用命令行模式进入容器。mysql-8-dev 就是上面设置的容器的名字。也可以用 `docker ps` 命令中显示的容器的 id（CONTAINER ID）。

```shell
> docker exec -it mysql-8-dev /bin/bash

bash-4.4# 
```

这里直接使用用户名和密码访问 mysql。

```shell
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

这样就表示成功进到 mysql 里面了。
