---
draft: false
create_date: 2021-06-29 08:00:00 +0800
date: 2021-06-29 08:00:00 +0800
title: "使用 Docker 启动 EasySwoole"
summary: "拉取容器；启动容器；异常处理；"
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
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- Docker v20.10.17

## 正文

### 拉取 Swoole 镜像

想启动 EasySwoole 框架需要先安装 Swoole 扩展。这里直接用 Swoole 提供的 docker 镜像。Swoole 的文档中就有 [链接](https://hub.docker.com/r/phpswoole/swoole)。

EasySwoole 官方文档中提到："如果没有特殊需求，请选择最新稳定版本开始下载(我这里是稳定版v4.4.23)"。所以，这里也使用 v4.4.23 版本的 Swoole。

```
> docker pull phpswoole/swoole:4.4.23-php7.4

4.4.23-php7.4: Pulling from phpswoole/swoole
...
Digest: sha256:5bd895677cbc73a06ea33239459bc4a07486d15025c1d1c805438a61c839dd32
Status: Downloaded newer image for phpswoole/swoole:4.4.23-php7.4
docker.io/phpswoole/swoole:4.4.23-php7.4
```

### 启动容器

```
> docker run -it -p 9501:9501 -v E:\code:/code phpswoole/swoole:4.4.23-php7.4 /bin/bash
```

- "-it" 参数加上最后的 "/bin/bash"，运行容器，并且以命令行模式进入容器。
- "-p 9501:9501"，将本机的 9501 端口和容器的 9501 端口进行映射。
- "-v E:\code:/code"，将本机的 "E:\code" 目录和容器的 "/code" 目录进行映射。

php 版本

```
> php -v
PHP 7.4.13 (cli) (built: Dec 18 2020 21:12:27) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
```

composer 版本

```
> composer -V
Composer version 1.10.19 2020-12-04 09:14:16
```

这个镜像安装了 pecl，可以用 pecl 安装需要的扩展

```
> pecl help version
PEAR Version: 1.10.12
PHP Version: 7.4.13
Zend Engine Version: 3.4.0
Running on: Linux 2bc0c5d254d5 5.10.25-linuxkit #1 SMP Tue Mar 23 09:27:39 UTC 2021 x86_64
```

### 异常处理

```
WARNING swManager_check_exit_status: worker#18[pid=642] abnormal exit, status=255,
```

遇到这样的报错是因为，swoole 和 xdebug 冲突了，需要关掉 docker 镜像中的 xdbug 扩展。

在这个镜像中，配置文件的路径是 "/usr/local/etc/php/conf.d/sdebug.ini-suggested"。把里面都注释掉就行了。
