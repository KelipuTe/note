---
draft: false
date: 2021-11-17 08:00:00 +0800
lastmod: 2021-11-17 08:00:00 +0800
title: "Git 常用命令"
summary: "Git 常用命令"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- application(应用)
- git
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> git 2.34.0

### 创建

`git init`，初始化 git 仓库。

初始化之后就会在当前目录创建 `.git/` 目录。

### 修改

`git add .`，添加所有文件到缓冲区。

`git status`，查看 git 库的状态。

当前在哪个分支，还有文件的变化情况，哪些文件被新增、修改、删除了。

`git diff`，用于文件修改了，但是还没有提交时，比较文件修改前后的差异。

`git diff`，默认比较全部的文件。

`git diff {filename}`，可以指定文件 filename 进行比较。

### 提交

`git commit -m "message"`，提交缓冲区的所有修改到本地仓库。

如果修改了但是没有 add 到缓冲区的，是不会被提交的。

`git reset`，回退版本。

`git reset HEAD^`，回退所有内容到上一个版本。

`git reset {code}`，回退所有内容到指定版本。

上面的 code 参数填的是 `git log` 输出的信息中，每个提交最前面的那一串。

`--hard` 参数，撤销工作区中所有未提交的修改内容，将暂存区与工作区都回退，并删除之前的所有的提交信息。

`git reset --hard HEAD^`，强行回退所有内容到上一个版本。

`git reset --hard {code}`，强行回退所有内容到指定版本。

`git reset --hard origin/master`，强行将本地分支回退到和远程 master 分支一样。

**谨慎使用 `--hard` 参数**。

### 日志

`git log`，查看历史记录，主要是提交信息。

`--oneline` 参数，查看历史记录的简洁的版本。

`--graph` 参数，查看历史记录中的分支和合并。

### 远程仓库

`git remote`，操作远程仓库

`git remote -v`，显示所有远程仓库。

`git remote add {name} {url}`，添加远程仓库 name 并设置地址为 url。

常用的：`git remote add origin {url}`，添加远程仓库 origin 并设置地址为 url。

`git remote set-url origin {url}`，修改远程仓库 origin 的地址为 url。如果未指定协议，则默认为 SSH。

`git remote rm {name}`，删除远程仓库 name 。

### 获取代码

`git fetch`，从远程仓库获取代码。

`git fetch origin`，从远程仓库 origin 获取代码。

### 获取代码并合并

`git pull`，从远程获取代码并合并。

`git pull` 其实是 `git fetch` 和 `git merge FETCH_HEAD` 的简写。

`git pull {name} {remote branch}:{locol branch}`，将远程仓库 name 的 remote branch 分支拉取过来，与本地的 locol branch 分支合并。

常用的：`git pull origin master`，将远程仓库 origin 的 master 分支拉取过来，与本地的当前分支合并。

### 上传代码并合并

`git push`,将本地的分支上传到远程并合并。

`git push {name} {remote branch}:{locol branch}`，将本地的 locol branch 分支推送到远程仓库 name 的 remote branch 分支。

如果本地的 locol branch 分支和远程仓库 name 的 remote branch 分支名字一样，可以简化成 `git push {name} {branch}`。

`git push origin master:master`，将本地的 master 分支推送到远程仓库 origin 的 master 分支。

常用的：`git push origin master` 是 `git push origin master:master` 的简写。

`-u` 参数，记录 push 到远端分支的默认值，下次想要继续 push 这个分支的时候，推送命令就可以简写成 `git push`。

执行 `git push -u origin master` 之后，下次在想推送 master 分支的时候，直接 `git push` 即可。

`--force` 参数，强制推送。

`git push --force master`，将本地的 master 分支强制推送到远程仓库 origin 的 master 分支并覆盖。

`git push --force master` 可以简写为 `git push -f master`。

**谨慎使用 `--force` 参数**。

`git push origin -d {name}`，删除远程仓库 origin 的 name 分支。

### 分支

`git branch`，列出本地分支。星号 `*` 标记当前在哪个分支下。

`git branch -a`，列出本地和远端的所有分支。星号 `*` 标记当前在哪个分支下。

`git branch {name}`，创建 name 分支。

`git branch -d {name}`，删除本地 name 分支。如果该分支有提交未进行合并，则会删除失败。

`git branch -D {name}`，强制删除本地 name 分支。如果该分支有提交未进行合并，也会删除成功。

`git checkout {name}`，切换到 name 分支。

`git checkout -b {name}`，创建 name 分支并切换到 name 分支。

`git checkout .`，把本地所有修改的，但是没有提交的，都回退到原来的状态。

### 合并

`git merge {name}`，合并 name 分支到当前分支。

如果合并时发生冲突，则会将有冲突的文件的文件名输出到命令行。

进入文件解决冲突后，需要执行 git add 命令通知 git 冲突已解决。

### 暂存

`git stash`，把所有没有提交的修改暂存。

`git stash pop`，恢复到 `git stash` 之前。

### 提交一个本地仓库到多个远程仓库

这里提交一个本地仓库到 Github 和 Gitee 两个远程仓库。

```
git remote add origin git@github.com:xxx/demo_project.git
git push origin master

git remote add gitee git@gitee.com:xxx/demo_project.git
git push gitee master
```
