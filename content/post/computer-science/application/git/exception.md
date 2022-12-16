---
draft: false
date: 2021-11-17 08:00:00 +0800
lastmod: 2021-11-17 08:00:00 +0800
title: "使用 Git 时遇到的一些异常和解决方案"
summary: "使用 Git 时遇到的一些异常和解决方案"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- Git
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> git 2.34.0

### 无法拉取代码

使用 `git pull` 命令拉取代码时卡住，并输出如下信息：

```
> git pull
Auto packing the repository in background for optimum performance.
See "git help gc" for manual housekeeping.
fatal: failed to run repack
```

这是因为，git 本地仓库如果长时间不进行清理，会导致本地 dangling commit 太多，从而造成拉取代码失败。

可以使用 `git fsck` 命令查看本地的 dangling commit。

```
> git fsck --lost-found

Checking object directories: 100% (256/256), done.
Checking objects: 100% (110393/110393), done.
dangling commit 01039c0c1efcc232be342aacc928af72df82503b
dangling blob 5b0bd062917d784486756ef941a7446b1aa5e340
dangling commit 4a1c18fa0803db6d64e9cffded496beb686aeaa7
dangling commit f528f06bd9739511bf962e32636697f807764cf8
dangling commit ea5fb0909b71b997c3feb5c436a150f745e40886
dangling commit c76448818640c1b0a3c00546abf16e5b4dff94e4
dangling commit d27fd8f73e071d3a629348f40d079888e821b10f
dangling commit 9a84703a10972d28e784d26417d6f53193b7ef78
dangling blob e7841cedb8d8a5b2cb02814cd9a2af06c5f46e00
dangling commit 6e8cf4714bc2ebb4d866a0aa046766598a1bcaf6
dangling blob fa990caae09e8d23923aed1d303e56272f6c8587
dangling commit 269e6c37f2208841d904577cd6e70d36f8ab0283
dangling commit 1ca1f0e7aafa762a675076fbc25ff6ff7048e41b
...
```

使用 `git gc` 命令清理后，就可以正常拉取代码了。

```
> git gc --prune=now

Enumerating objects: 93146, done.
Counting objects: 100% (93146/93146), done.
Delta compression using up to 12 threads
Compressing objects: 100% (28200/28200), done.
Writing objects: 100% (93146/93146), done.
Total 93146 (delta 67637), reused 88951 (delta 64013), pack-reused 0
Removing duplicate objects: 100% (256/256), done.
```
