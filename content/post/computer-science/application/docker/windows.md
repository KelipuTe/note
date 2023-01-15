---
draft: false
date: 2021-06-29 08:00:00 +0800
lastmod: 2021-06-29 08:00:00 +0800
title: "Windows 10 环境使用 Docker"
summary: "安装，镜像，使用，异常处理"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- application(应用)
- docker
---

### 安装

去官网下一个 win10 使用的 Docker Desktop（下文简称 desktop）安装包。因为 Docker Desktop For Windows 使用的是 hyper-v 虚拟机，所以安装之前需要启用 win10 系统中内置的 hyper-v 虚拟机服务。

安装之后，可以打开 win10 的 powershell，执命令行 `docker run hello-world`，检测是否安装成功。安装成功的话，输出应该是下面这样的。

```powershell
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

### 镜像加速

国内从 DockerHub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。

打开 desktop 的界面，点击右上角的设置按钮（齿轮图标）打开设置界面。然后在左侧选择 Docker Engine 标签，可以看见 json 格式的配置参数。

```json
{
  "registry-mirrors": [
    "https://reg-mirror.qiniu.com"
  ],
  "insecure-registries": [],
  "debug": false,
  "experimental": false,
  "features": {
    "buildkit": true
  },
  "builder": {
    "gc": {
      "enabled": true,
      "defaultKeepStorage": "20GB"
    }
  }
}
```

在 registry-mirrors 参数中添加镜像加速的地址，这里用的是七牛云的加速器 `https://reg-mirror.qiniu.com`。

### 异常处理

```powershell
Error response from daemon: status code not OK but 500
```

如果在使用目录映射时，遇到上述这样的错误，可能是因为没有给docker权限。

打开 desktop 的设置界面左侧 Resources 标签的 File Sharing 子标签。然后添加 docker 可以映射的目录。

### 查看容器

在 powershell 中，使用 `docker ps` 命令可以查看正在使用的容器。

- `-a` 参数可以查看所有的包括未启动的容器。

```powershell
> docker ps -a
```

### 进入容器

在 powershell 中，使用 `docker exec` 命令可以使用命令行模式进入容器。

- container id，容器 id。

```powershell
> docker exec -it {container id} /bin/bash
```

### 导出容器

在 powershell 中，使用 `docker export` 命令导出容器。建议用第二个命令，在 powershell 中第一个可能有 bug。

- container id，容器 id。
- file path，导出文件的路径和名字，比如 e:\xxx.tar。
- `>` 和 `-o` 表示导出到文件。

```powershell
> docker export {container id} > {file path}
> docker export -o {file path} {container id}
```

