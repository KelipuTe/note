---
draft: false
date: 2021-06-29 08:00:00 +0800
lastmod: 2021-06-29 08:00:00 +0800
title: "使用 Docker 部署 EasySwoole"
summary: "使用 Docker 部署 EasySwoole"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- application(应用)
- docker
- php
- EasySwoole
---

### 部署 Swoole

Swoole 官方文档中提供了官方的 docker 镜像 `https://hub.docker.com/r/phpswoole/swoole`。

EasySwoole 官方文档中提到："如果没有特殊需求，请选择最新稳定版本开始下载(我这里是稳定版v4.4.23)"。所以这里也使用 v4.4.23 版本的 Swoole。

DockerHub 页面右侧提供的命令 `docker pull phpswoole/swoole` 会默认拉取最新的镜像，这里我们需要指定版本的所以不能用上面的命令。

我们需要点击页面上的 Tags 标签，然后在下面找到 php7.4 版本 swoole 4.4.23 版本的 tag，tag 右侧提供了拉取镜像的命令 `docker pull phpswoole/swoole:4.4.23-php7.4`。打开 PowerShell 执命令行，拉取成功的话会是下面这样的输出。

```powershell
> docker pull phpswoole/swoole:4.4.23-php7.4
4.4.23-php7.4: Pulling from phpswoole/swoole
6ec7b7d162b2: Pull complete
db606474d60c: Pull complete
afb30f0cd8e0: Pull complete
3bb2e8051594: Pull complete
e2b7fe41b468: Pull complete
9ab9906ce2f0: Pull complete
50fac8a5156f: Pull complete
1029765b1cd3: Pull complete
e9a21d7557c6: Pull complete
517180811701: Pull complete
b9c164c9c98f: Pull complete
Digest: sha256:5bd895677cbc73a06ea33239459bc4a07486d15025c1d1c805438a61c839dd32
Status: Downloaded newer image for phpswoole/swoole:4.4.23-php7.4
docker.io/phpswoole/swoole:4.4.23-php7.4
```

同时使用命令 `docker images` 也能看到可以使用的镜像列表。

```powershell
> docker images
REPOSITORY         TAG             IMAGE ID       CREATED        SIZE
hello-world        latest          d1165f221234   3 months ago   13.3kB
phpswoole/swoole   4.4.23-php7.4   fb33e322751b   5 months ago   480MB
```

### 启动容器

```powershell
> docker run -it -p 127.0.0.1:9503:9503 -v E:\code:/code phpswoole/swoole:4.4.23-php7.4 /bin/bash
```

这个命令，将本机 9503 端口映射到容器 9503 端口，将本机 E:\code 目录映射到容器 /code 目录。

php版本

```shell
> php -v
PHP 7.4.13 (cli) (built: Dec 18 2020 21:12:27) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
```

composer版本

```shell
> composer -V
Composer version 1.10.19 2020-12-04 09:14:16
```

这个镜像安装了 pecl，可以用 pecl 安装需要的扩展

```shell
> pecl help version
PEAR Version: 1.10.12
PHP Version: 7.4.13
Zend Engine Version: 3.4.0
Running on: Linux 2bc0c5d254d5 5.10.25-linuxkit #1 SMP Tue Mar 23 09:27:39 UTC 2021 x86_64
```

### 异常处理

```shell
WARNING swManager_check_exit_status: worker#18[pid=642] abnormal exit, status=255,
```

遇到这样的报错是因为 swoole 和 xdebug 冲突了，需要关掉官方提供的 docker 镜像中的 xdbug 扩展。php7.4 版本 swoole 4.4.23 版本的镜像中的配置文件在 `/usr/local/etc/php/conf.d/sdebug.ini-suggested`。把里面都注释掉就行了。