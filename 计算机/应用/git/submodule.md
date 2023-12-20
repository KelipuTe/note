---
draft: false
date: 2023-03-30 08:00:00 +0800
lastmod: 2023-03-30 08:00:00 +0800
title: "submodule"
summary: "误删 submodule 的目录；"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- application(应用)
- git
---
## 正文

### 删除 submodule 目录后，重新执行命令添加的时候报错怎么办

在使用 hugo 的时候，会使用 submodule 来添加主题。误删 submodule 的目录，重新执行命令添加的时候报如下的错误。

```
> git submodule add https://github.com/hugo-next/hugo-theme-next.git themes/hugo-theme-next
fatal: 'themes/hugo-theme-next' already exists in the index
```

解决方法是：

- 删掉 submodule 的目录。（这里已经被删掉了）
- 打开 {项目根目录}/.gitmodules，找到对应的 submodule 条目，删掉。
- 打开 {项目根目录}/.git/config，找到对应的 submodule 条目，删掉。
- 删掉 {项目根目录}/.git/modules/{对应的目录}。

然后就可以重新执行命令添加了。

## 参考（reference）

- [git submodule删除后重新添加问题](https://blog.csdn.net/dongguanghuiyin/article/details/78792992)
