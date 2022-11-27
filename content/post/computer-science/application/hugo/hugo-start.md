---
draft: false
date: 2022-09-20 08:00:00 +0800
lastmod: 2022-09-20 08:00:00 +0800
title: "使用 Hugo 和 GitHub Pages 搭建站点"
summary: "Hugo 安装，建立站点，配置主题，发布到 GitHub Pages 的过程。"

categories:
- application(应用)
  
tags:
- computer-science(计算机科学)
- Hugo
- GitHub Pages
---

- Windows 11 家庭版
- cpu amd64
- hugo 0.103.1

### Hugo 文档

- [gohugoio/hugo](https://github.com/gohugoio/hugo)
- [英文文档](https://gohugo.io/)
- [中文文档](https://www.gohugo.org/)

### 安装 Hugo

在 [Hugo Releases](https://github.com/gohugoio/hugo/releases) 页面下载对应操作系统的版本。这里下载的是 hugo_0.103.1_windows-amd64.zip。

下载完成后，解压到想要的位置。这里使用的目录是 **D:\hugo\bin**。然后将这个目录添加到 **系统变量 path** 中（我的电脑 -> 属性 -> 高级系统设置 -> 环境变量 -> 系统变量 -> path）。

搞定之后，可以打开控制台，输出一下版本信息，验证一下安装是否成功。或者试试 `hugo help` 命令，看看能不能输出帮助信息。

```shell
> hugo version
hugo v0.103.1-b665f1e8f16bf043b9d3c087a60866159d71b48d windows/amd64 BuildDate=2022-09-18T13:19:01Z VendorInfo=gohugoio
```

如果有需要的话，需要安装 extended 版本的。这里下载的是 hugo_extended_0.103.1_windows-amd64.zip。

```shell
> hugo version
hugo v0.103.1-b665f1e8f16bf043b9d3c087a60866159d71b48d+extended windows/amd64 BuildDate=2022-09-18T13:19:01Z VendorInfo=gohugoio
```

### 创建站点

使用命令创建一个站点，如果没问题的话，hugo 会在当前目录下创建一个名字是 project-name 的目录。

```shell
> hugo new site {project-name}
```

新建的站点没有任何内容，可以使用命令创建一个内容页面。新创建的文件会在目录 **content/** 里。创建内容页面的时候也可以带上目录。

```shell
> hugo new helloworld.md
> hugo new posts/helloworld.md
```

### 安装主题

这里就是和别的站点工具不一样的地方了，比如 hexo 和 jekyll，如果没有安装主题的话，是会有一个默认的主题的。

但是 hugo 没有默认主题，需要去主题库下载一个，然后添加到站点里并配置好，这样才能启动站点。如果没有安装主题就启动的话，会报没有模板的错误。

可以去 [官方的主题库](https://themes.gohugo.io/) 找一个喜欢的。然后按照主题提供的文档配置一下。

#### Hugo NexT

[Hugo NexT](https://themes.gohugo.io/themes/hugo-theme-next) 这个主题是从 Hexo NexT 移植过来的。

GitHub 项目地址 [hugo-next/hugo-theme-next](https://github.com/hugo-next/hugo-theme-next)。

按着 Hexo NexT 主题提供的的文档走，把主题配置到站点中。

别忘了先 `git init`，然后使用命令下载主题 `git submodule add https://github.com/hugo-next/hugo-theme-next.git themes/hugo-theme-next`。

如果需要升级主题的话，就进入 **{path-to-project}/themes/github-style** 目录，执行 `git pull` 命令，拉取最新的代码。

然后把 **{path-to-project}themes/hugo-theme-next/exampleSite/** 目录下所有的文件复制到 **{path-to-project}/** 目录下覆盖。

最后删除原来的配置文件 config.toml，然后就可以使用命令 `hugo server` 启动服务了。

另外需要注意的是，这个主题需要 hugo extended 版本，如果用的不是 extended 版本，启动的时候会报下面这样的错，提示去安装 extended 版本。

```shell
> hugo server
Start building sites …
hugo v0.103.1-b665f1e8f16bf043b9d3c087a60866159d71b48d windows/amd64 BuildDate=2022-09-18T13:19:01Z VendorInfo=gohugoio
WARN 2022/09/20 21:12:38 Hugo NexT 主题使用了 SCSS 框架，请到官方地址下载 Hugo Extended 版本：https://github.com/gohugoio/hugo/releases
ERROR 2022/09/20 21:12:38 Because that use SCSS framework in Hugo NexT, Please download Hugo extended version on offical site: https://github.com/gohugoio/hugo/releases
Error: Error building site: TOCSS: failed to transform "main.scss" (text/x-scss). Check your Hugo installation; you need the extended version to build SCSS/SASS.: this feature is not available in your current Hugo version, see https://goo.gl/YMrWcn for more information
```

#### github-style（选看）

[github-style](https://themes.gohugo.io/themes/github-style/) 这个主题是 github 的页面风格。

GitHub 项目地址 [MeiK2333/github-style](https://github.com/MeiK2333/github-style)。

按着 github-style 主题提供的的文档走，把主题配置到站点中。

别忘了先 `git init`，然后使用命令下载主题 `git submodule add git@github.com:MeiK2333/github-style.git themes/github-style`。

如果需要升级主题的话，就进入 **{path-to-project}/themes/github-style** 目录，执行 `git pull` 命令，拉取最新的代码。

在 **content/** 里创建 **post/** 目录，后面所有的内容页面都放到这个目录下面，要不然站点里不会展示。

最后在配置文件 config.toml 里设置主题 `theme = "github-style"`。然后就可以使用命令 `hugo server` 启动服务了。

站点是没有问题的，可以正常地跑起来。但是这个主题貌似没有实现标签分类，也可能是本人没有找到，所以就放弃继续使用了。

### 部署到 GitHub Pages

使用命令 `hugo -t {theme-name}` 来把发布用的目录编译出来。这里就是 `hugo -t hugo-theme-next`。

默认情况下会编译到 **{path-to-project}/publish/** 目录。 可以通过编辑配置文件，在配置文件里添加 `publishDir: docs`，来修改这个目录。

push 到 github.io 的时候，如果使用的是 **publish/** 目录。那么要 push **publish/** 目录上去，然后设置 GitHub Pages 的 Branch 为 **master** 和 **/(root)**。

如果使用的是 **docs/** 目录，那么就要 push 整个项目上去，然后设置 GitHub Pages 的 Branch 为 **master** 和 **docs/**。

### 文本头部信息

---
draft: true
date: 2000-01-01 08:00:00 +0800
lastmod: 2002-01-01 08:00:00 +0800
title: "title"
summary: "summary"

categories:
- categories(分类)

tags:
- tags1(标签1)
- tags2(标签2)
---

- draft：是不是草稿，true=是；false=不是。启动的时候带 `--buildDrafts` 选项就可以看到草稿的内容。
- date：创建时间
- lastmod：最后修改时间
- title：文本标题
- summary：文本概述 
- categories：文本分类，一般一个 
- tags：文本标签，可以多个

### reference（参考）

- [hugo个人博客搭建并部署到GitHub【 for Windows】](https://www.jianshu.com/p/cc73559fea2c)
