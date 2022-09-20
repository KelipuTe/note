---
title: "使用 Jekyll 主题"
date: 2022-04-28T20:00:00+08:00
draft: false
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
Fetching gem metadata from https://rubygems.org/..........
Resolving dependencies...
Fetching rake 10.5.0
Installing rake 10.5.0
Using bundler 2.3.7
Using racc 1.6.0
Fetching concurrent-ruby 1.1.10
Fetching minitest 5.15.0
Fetching public_suffix 4.0.7
Fetching eventmachine 1.2.7
Fetching colorator 1.1.0
Fetching http_parser.rb 0.8.0
Fetching ffi 1.15.5 (x64-mingw-ucrt)
Fetching forwardable-extended 2.6.0
Fetching gemoji 3.0.1
Fetching rb-fsevent 0.11.1
Fetching rexml 3.2.5
Fetching liquid 4.0.3
Fetching rouge 3.28.0
Fetching mercenary 0.4.0
Fetching safe_yaml 1.0.5
Fetching unicode-display_width 1.8.0
Installing safe_yaml 1.0.5
Fetching jekyll-paginate 1.1.0
Installing unicode-display_width 1.8.0
Fetching nokogiri 1.13.4 (x64-mingw-ucrt)
Installing liquid 4.0.3
Installing rexml 3.2.5
Fetching terminal-table 2.0.0
Fetching kramdown 2.4.0
Installing gemoji 3.0.1
Installing rb-fsevent 0.11.1
Installing colorator 1.1.0
Installing mercenary 0.4.0
Installing jekyll-paginate 1.1.0
Installing http_parser.rb 0.8.0 with native extensions
Installing terminal-table 2.0.0
Installing kramdown 2.4.0
Installing ffi 1.15.5 (x64-mingw-ucrt)
Fetching sassc 2.4.0
Fetching rb-inotify 0.10.1
Installing rb-inotify 0.10.1
Fetching listen 3.7.1
Fetching kramdown-parser-gfm 1.1.0
Installing listen 3.7.1
Installing forwardable-extended 2.6.0
Fetching pathutil 0.16.2
Installing kramdown-parser-gfm 1.1.0
Fetching jekyll-watch 2.2.1
Installing jekyll-watch 2.2.1
Installing concurrent-ruby 1.1.10
Installing pathutil 0.16.2
Fetching i18n 1.10.0
Fetching tzinfo 2.0.4
Installing minitest 5.15.0
Installing eventmachine 1.2.7 with native extensions
Installing tzinfo 2.0.4
Installing sassc 2.4.0 with native extensions
Installing i18n 1.10.0
Fetching activesupport 7.0.2.4
Installing activesupport 7.0.2.4
Installing public_suffix 4.0.7
Fetching addressable 2.8.0
Installing addressable 2.8.0
Installing rouge 3.28.0
Fetching em-websocket 0.5.3
Installing em-websocket 0.5.3
Installing nokogiri 1.13.4 (x64-mingw-ucrt)
Fetching html-pipeline 2.14.1
Installing html-pipeline 2.14.1
Fetching jekyll-sass-converter 2.2.0
Installing jekyll-sass-converter 2.2.0
Fetching jekyll 4.2.2
Installing jekyll 4.2.2
Fetching jekyll-feed 0.16.0
Fetching jekyll-sitemap 1.4.0
Fetching jemoji 0.12.0
Installing jemoji 0.12.0
Installing jekyll-sitemap 1.4.0
Installing jekyll-feed 0.16.0
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
