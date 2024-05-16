---
draft: false
title: "ZooKeeper 怎么用"
summary: "怎么在 docker 里面启动；怎么在 docker 里面启动；怎么在 golang 里面使用；"
toc: true

categories:
  - 组件

tags:
  - 计算机
  - 组件
  - zookeeper

date: 2024-05-15 08:00:00 +0800
---

## 资料

环境参数：

- ZooKeeper 3.9
- go1.20
- go-zookeeper/zk v1.0.3

代码：{demo-golang}/demo/zookeeper/

## 正文

### 怎么在 docker 里面启动

DockerHub 里的 ZooKeeper 镜像。
[链接](https://hub.docker.com/_/zookeeper) 在这，可能需要梯子。

查询 zookeeper 有哪些 docker 镜像可以用。

```
> docker search zookeeper
NAME                             DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
zookeeper                        Apache ZooKeeper is an open-source server wh…   1425      [OK]
#列表里面还有很多条目，这里我们就用第一条目。
```

下载 docker 镜像。这里下载的是 3.9 版本的。

```
> docker pull zookeeper:3.9
3.9: Pulling from library/zookeeper
#这里会去下载 docker 镜像，需要等一会。成功下载下来之后，会输出下面这些内容。
4a023cab5400: Pull complete
dce394e5c05f: Pull complete
e026920ea2a9: Pull complete
70d0df997d9c: Pull complete
c863a9ccf459: Pull complete
0076f3e1a3ca: Pull complete
24cc5b28d86d: Pull complete
4106501f119e: Pull complete
d5abc756bc3a: Pull complete
Digest: sha256:e17ab786978ac37d6e06a4b756b0c63e9dae725016056dd0b96286c6bb3f1adb
Status: Downloaded newer image for zookeeper:3.9
docker.io/library/zookeeper:3.9
```

docker 镜像下载完成之后，就可以在 docker images 命令的列表里面看到了。

```
> docker images
REPOSITORY             TAG       IMAGE ID       CREATED         SIZE
zookeeper              3.9       68f009cf243c   12 days ago     314MB
#列表里面还有很多条目，这里我们就看 zookeeper 的条目。
```

然后就可以用 docker run 命令启动容器了。

```
> docker run \
-d \
-e TZ="Asia/Shanghai" \
-p 2181:2181 \
-v D:\tmp\docker\zookeeper-single:/root/docket/zookeeper \
--name zookeeper-single \
--restart always \
zookeeper:3.9
```

### 怎么在 docker 里面使用

进入运行中的容器。

```
> docker exec -it zookeeper-single /bin/bash
```

进去之后直接在 zookeeper 的根目录。
进入 bin 文件夹，里面有一些 shell 脚本可以用。

```
> root@880f3b90bc07:/apache-zookeeper-3.9.2-bin# cd bin/
> root@880f3b90bc07:/apache-zookeeper-3.9.2-bin/bin# ll
total 88
drwxr-xr-x 2 zookeeper zookeeper  4096 Feb 12 20:48 ./
drwxr-xr-x 6 zookeeper zookeeper  4096 May  2 07:59 ../
-rwxr-xr-x 1 zookeeper zookeeper   232 Feb 12 20:48 README.txt*
-rwxr-xr-x 1 zookeeper zookeeper  1978 Feb 12 20:48 zkCleanup.sh*
-rwxr-xr-x 1 zookeeper zookeeper  1115 Feb 12 20:48 zkCli.cmd*
-rwxr-xr-x 1 zookeeper zookeeper  1576 Feb 12 20:48 zkCli.sh*
-rwxr-xr-x 1 zookeeper zookeeper  1810 Feb 12 20:48 zkEnv.cmd*
-rwxr-xr-x 1 zookeeper zookeeper  3947 Feb 12 20:48 zkEnv.sh*
-rwxr-xr-x 1 zookeeper zookeeper  1243 Feb 12 20:48 zkServer.cmd*
-rwxr-xr-x 1 zookeeper zookeeper  4559 Feb 12 20:48 zkServer-initialize.sh*
-rwxr-xr-x 1 zookeeper zookeeper 11680 Feb 12 20:48 zkServer.sh*
-rwxr-xr-x 1 zookeeper zookeeper   987 Feb 12 20:48 zkSnapshotComparer.cmd*
-rwxr-xr-x 1 zookeeper zookeeper  1374 Feb 12 20:48 zkSnapshotComparer.sh*
-rwxr-xr-x 1 zookeeper zookeeper   995 Feb 12 20:48 zkSnapshotRecursiveSummaryToolkit.cmd*
-rwxr-xr-x 1 zookeeper zookeeper  1422 Feb 12 20:48 zkSnapshotRecursiveSummaryToolkit.sh*
-rwxr-xr-x 1 zookeeper zookeeper   988 Feb 12 20:48 zkSnapShotToolkit.cmd*
-rwxr-xr-x 1 zookeeper zookeeper  1377 Feb 12 20:48 zkSnapShotToolkit.sh*
-rwxr-xr-x 1 zookeeper zookeeper   996 Feb 12 20:48 zkTxnLogToolkit.cmd*
-rwxr-xr-x 1 zookeeper zookeeper  1385 Feb 12 20:48 zkTxnLogToolkit.sh*
```

zkServer.sh 脚本是用来启动服务的。
这里是不需要的，上面在启动容器的时候，服务已经启动了。

zkCli.sh 脚本可以打开一个连接到服务的终端。
在这个终端里面就可以执行 zookeeper 的命令了。

### zookeeper 命令

在线文档的 ZooKeeper CLI 这篇里讲了命令怎么用。
在线文档的 [链接](https://zookeeper.apache.org/)。
ZooKeeper CLI 这篇的 [链接](https://zookeeper.apache.org/doc/current/zookeeperCLI.html)。

`ls {path}`。
查看 {path} 路径对应的结点的子结点。

`create [-e] [-s] {path} {data} {acl}`。
创建路径为 {path} 的结点。{data} 是要存储的数据。{acl} 是访问权限，默认是 world。
`-e` 参数可选，代表临时节点。临时节点不能创建子节点。`-s` 参数可选，代表顺序节点。
`-e` 和 `-s` 可以一起使用，没有 `-e` 和 `-s` 就是永久节点。

`delete {path}`。
删除 {path} 路径对应的结点。

`deleteall {path}`
删除 {path} 路径下面所有的结点。

`get [-w] {path}`。
获取 {path} 路径对应的结点的数据。
`-w` 参数可选，表示持续监听。

`set [-v {version}] {path} {data} `。
修改 {path} 路径对应的结点的数据。
-v 参数可选，结点的 version 和命令中的 version 是一样的时候才会执行修改，可用作乐观锁。

version 不匹配就会报错，像下面这样。

```
> set [-v {version}] {path} {data}
version No is not valid : {path}
```

`stat [-w] {path}`。
获取 {path} 路径对应的结点的状态信息。
`-w` 参数可选，表示持续监听。

`quit`。退出 cli 终端。

### 怎么在 golang 里面使用

go mod 添加依赖。

```
> go get github.com/go-zookeeper/zk
go: downloading github.com/go-zookeeper/zk v1.0.3
go: added github.com/go-zookeeper/zk v1.0.3
```

**详细示例见：{demo-golang}/demo/zookeeper/**
