---
draft: false
title: "Windows 10 环境使用 Docker"
summary: "安装；镜像加速；异常处理；如何使用；"
toc: true

categories:
  - application(应用)

tags:
  - computer-science(计算机科学)
  - application(应用)
  - docker

date: 2023-05-09 08:00:00 +0800
---

## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- Docker v20.10.17

## 正文

### 安装

去 [Docker 官网](https://www.docker.com/) 下一个 Windows 11 使用的 Docker Desktop For Windows（下面简称 docker） 安装包。

因为，Docker Desktop For Windows 使用的是 hyper-v 虚拟机。所以，安装之前，需要安装并启用 Windows 11 的 hyper-v 虚拟机服务。

安装之后，可以打开 Windows PowerShell（下面简称 powershell），执命令行 `docker run hello-world`，检测是否安装成功。成功的话，输出应该是下面这样的。

```
> docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### 输出版本信息

也可以通过，在 powershell 中，使用 "docker version" 命令，输出 docker 的版本信息，检测是否安装成功。这里截取了部分输出。

```
> docker version

Client:
 Version:           20.10.17
 Go version:        go1.17.11
 OS/Arch:           windows/amd64
```

### 镜像加速

国内从 [DockerHub](https://hub.docker.com) 拉取镜像有时会遇到困难，此时可以配置镜像加速器。

打开 docker 的界面；点击右上角的 "设置按钮"（图标是个齿轮），打开 "设置界面"；
点击左侧的 "Docker Engine 标签"，就可以看见 json
格式的配置参数。

在 registry-mirrors 参数中添加镜像加速的地址，这里用的是七牛云的加速器 "https://reg-mirror.qiniu.com"。
如果没有
registry-mirrors 参数的话，就加一个。

修改好之后，点击右下角的 "Apply & Restart 按钮"，等 docker 转完圈应该就可以了。如果不行的话，重启 docker 试试。

```json
{
  "registry-mirrors": [
    "https://reg-mirror.qiniu.com"
  ]
}
```

### 输出 docker 的信息

在 powershell 中，使用 "docker info" 命令，输出 docker 的信息。

如果操作了上面的镜像加速的部分，这里在输出理就可以看到刚才设置的加速器的地址。

```
> docker info

Registry Mirrors:
  https://reg-mirror.qiniu.com/
```

### 异常处理

```
Error response from daemon: status code not OK but 500
```

如果在使用目录映射时，遇到上面这样的报错，那可能是因为没有给 docker 权限。

打开设置界面；点击左侧的 "Resources 标签"；点击 "File Sharing 子标签"。然后添加 docker 可以映射的目录。

```
> docker ps -a
```

### 拉取镜像和启动容器

- [mysql](/计算机/application/docker/mysql)
- [docker compose](/计算机/application/docker/compose)
- [rabbitmq](/计算机/application/docker/rabbitmq)
- [easyswoole](/计算机/application/docker/easyswoole)

### 查看容器

在 powershell 中，使用 "docker ps" 命令，可以查看正在使用的容器。

- "-a" 参数，可以查看所有的包括未启动的容器。

### 进入容器

容器启动之后，在 powershell 中，使用 "docker exec" 命令，可以使用命令行模式进入容器。

```
> docker exec -it {container} /bin/bash
```

- {container}，可以填容器的名字或者容器的 ID（"docker ps" 命令中输出的 CONTAINER ID）。

### 导出容器

在 powershell 中，使用 "docker export" 命令，可以导出容器。

```
> docker export {container} > {file path}
> docker export -o {file path} {container}
```

- {container}，可以填容器 id。
- {file path}，导出文件的路径和名字。比如，e:\xxx.tar。
- ">" 操作符和 "-o" 参数，都表示导出到文件。

建议用第二个命令，在 powershell 中第一个可能有 bug。

docker run 参数
`--name {容器名字}` 自定义容器名字
`-p {本机端口}:{容器端口}` 映射端口
`-d` 后台运行
--restart always 每次都重启
-e TZ="Asia/Shanghai" 指定时区
-v {D:\tmp\docker\zookeeper-single}:{/root/docket/zookeeper} 映射目录