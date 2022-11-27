---
draft: false
date: 2022-04-28 08:00:00 +0800
lastmod: 2022-04-28 08:00:00 +0800
title: "使用 Jekyll 主题"
summary: "Jekyll 配置主题的过程。"

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- Ruby
- Jekyll
- GitHub Pages
---

### 环境

- Windows 11 家庭版
- Ruby 3.1.2
- RubyGems 3.3.7
- Jekyll 4.2.2

### jekyll-TeXt-theme

Github：[https://github.com/kitian616/jekyll-TeXt-theme](https://github.com/kitian616/jekyll-TeXt-theme)

官方文档：[https://tianqi.name/jekyll-TeXt-theme/](https://tianqi.name/jekyll-TeXt-theme/)

第 1 步，从 Github 上把项目下载下来。

第 2 步，安装 Ruby 依赖包。

```
> bundle install --path vendor/bundle

[DEPRECATED] The `--path` flag is deprecated because it relies on being remembered across bundler invocations, which bundler will no longer do in future versions. Instead please use `bundle config set --local path 'vendor/bundle'`, and stop using this flag
一大堆输出。。。
Using jekyll-text-theme 2.2.6 from source at `.`
Bundle complete! 3 Gemfile dependencies, 41 gems now installed.
Bundled gems are installed into `./vendor/bundle`
Post-install message from html-pipeline:
-------------------------------------------------
Thank you for installing html-pipeline!
You must bundle Filter gem dependencies.
See html-pipeline README.md for more details.
https://github.com/jch/html-pipeline#dependencies
-------------------------------------------------
```

安装完成后，可以使用 Jekyll 集成的那个开发用的服务器，然后使用浏览器在本地进行预览。

这里同样会遇到 webrick 无法加载的那个异常情况，解决方法是一样的。

### jekyll-theme-chirpy

Github：[https://github.com/cotes2020/jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)

官方文档（同时也是个 Demo）：[https://chirpy.cotes.page/](https://chirpy.cotes.page/)

个人感觉，这个主题比上面那个正在用的要好看不少。

但是无奈，按照官方文档走流程的时候，在发布流程的 Github Action 步骤卡住了。而且没有找到解决方案，遂遗憾放弃。
