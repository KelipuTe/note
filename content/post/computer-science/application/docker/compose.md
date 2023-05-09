---
draft: false
create_date: 2023-01-08 08:00:00 +0800
date: 2023-01-08 08:00:00 +0800
title: "Docker Compose 怎么用"
summary: "编辑 docker-compose.yaml；启动 Docker Compose；"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- application(应用)
- docker
- docker-compose
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- Docker v20.10.17

## 正文

### Docker Compose 是什么

Docker Compose 是用于定义和运行多个 Docker 容器的工具。
使用 YML 文件配置需要的所有服务后，可以同时创建并启动所有服务。

### Docker Compose 怎么用

#### 第一步：编辑 docker-compose.yaml

```
services:
  mysql8:
    image: mysql:8.0
    container_name: mysql8-dev
#    restart: always
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_ROOT_PASSWORD: root
#    volumes:
#      - ./script/mysql/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"

  redis7:
    image: redis:7.0
    container_name: redis7-dev
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - '6379:6379'

```

- image：指定用哪个 docker 镜像，如果本地没有的话，启动的时候会去 DockerHub 拉。
- container_name：指定容器名称。
  如果不指定的话，当多个 docker compose 都需要创建同一个容器的时候，会报重名的错误。
  如果指定的话，多个 docker compose 会共享那个指定了名称的容器。
- restart：需不需要重启容器。
- ports：指定端口映射。冒号前面的是本机的端口，冒号后面的是容器内的端口。

#### 第二步：启动

打开 powershell，进入 docker-compose.yaml 文件所在的目录。

执行 `docker compose up` 或者 `docker compose up -v` 命令就可以启动了。

带 `-v` 参数表示命令在后台运行，不会再控制台输出日志。想要关闭的时候直接 Ctrl+C 就可以了。
