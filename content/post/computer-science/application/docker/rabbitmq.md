---
draft: false
create_date: 2022-05-18 08:00:00 +0800
date: 2022-10-17 08:00:00 +0800
title: "使用 Docker 启动 MySQL"
summary: "拉取容器；启动容器；RabbitMQ 的 Web 管理界面；"
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

没啥特别的要求，这里直接拉最新的 RabbitMQ 镜像。

```
> docker pull rabbitmq

latest: Pulling from library/rabbitmq
Digest: sha256:a64d81498c47681cb797ce56c775a95aab751ebc21e90f007fdb45553e391cf9
Status: Image is up to date for rabbitmq:latest
docker.io/library/rabbitmq:latest
```

### 启动容器

```
docker run -itd --name rabbitmq-dev -p 5672:5672 -p 15672:15672 rabbitmq
```

- "-p 5672:5672 -p 15672:15672"，注意这里有两个端口，5672 是服务的端口，15672 是 Web 管理界面的端口。

### RabbitMQ 的 Web 管理界面

管理界面默认不是开启的，需要进入容器，使用下面的命令启用管理插件。

```
> rabbitmq-plugins enable rabbitmq_management

Enabling plugins on node rabbit@c47a12f3024b:
rabbitmq_management
The following plugins have been configured:
rabbitmq_delayed_message_exchange
rabbitmq_management
rabbitmq_management_agent
rabbitmq_prometheus
rabbitmq_web_dispatch
Applying plugin configuration to rabbit@c47a12f3024b…
The following plugins have been enabled:
rabbitmq_management

started 1 plugins.
```

插件激活后，无需重新启动节点，直接就可以通过浏览器进行访问了。默认的用户名和密码都是 "guest"。
