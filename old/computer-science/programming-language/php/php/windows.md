## windows环境安装和配置php

- 2021-11-17
- windows-10
- php-7.4.25

### 下载

- 去php手册首页（https://www.php.net/）。
- 在页面右侧download找到7.4.25并点击。
- 找到7.4.25版本对应的windows downloads并点击。
- 找到64位系统线程安全版本的（VC15 x64 Thread Safe (2021-Nov-16 20:36:58)）并下载对应的zip压缩文件。
- 本人下载下来的压缩文件叫php-7.4.25-Win32-vc15-x64.zip。
- 下载完成后解压到合适的位置。

### 配置

- 在桌面右击我的电脑找到属性并点击。
- 在界面左侧找到高级系统设置并点击。
- 在界面下面找到环境变量并点击。
- 在界面的系统变量列表中找到path变量并点击编辑。
- 在path变量中添加php根目录和php扩展目录。
- 本人的这两个目录是`C:\php\php7.4.25`和`C:\php\php7.4.25\ext`。

### php.ini

在php根目录下，原地复制一个php.ini-development，改名为php.ini。

用记事本打开php.ini，找到下面这一段：

```
; Directory in which the loadable extensions (modules) reside.
; http://php.net/extension-dir
;extension_dir = "./"
; On windows:
;extension_dir = "ext"
```

在后面加一段`extension_dir = "C:\php\php7.4.25\ext"`，当然直接改也可以。这里的extension_dir参数的值对应上面的php扩展目录。

### php版本

#### php -v

可以在win10的命令行窗口（cmd黑窗口），使用`php -v`命令查看php版本信息。

```
> C:\Users\Administrator>php -v
PHP 7.4.25 (cli) (built: Oct 20 2021 09:30:08) ( ZTS Visual C++ 2017 x64 )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
```

### php扩展

#### php -m

可以在命令行窗口，使用`php -m`命令查看php启用的扩展。

```
> C:\Users\Administrator>php -m
[PHP Modules]
bcmath
calendar
Core
ctype
date
dom
filter
hash
iconv
json
libxml
mysqlnd
pcre
PDO
Phar
readline
redis
Reflection
session
SimpleXML
SPL
standard
tokenizer
xml
xmlreader
xmlwriter
zip
zlib

[Zend Modules]
```

#### 添加扩展

这里用mysql扩展和redis扩展举例。

mysql扩展是自带的，可以在php.ini中找到这一行：`;extension=mysqli`。

如果前面有分号说明这行代码是被注释的状态，去掉最前面的分号就可以启用扩展。

redis扩展不是自带的。需要去pecl（https://pecl.php.net）上下载。

- 去pecl首页（https://pecl.php.net）。
- 在pecl首页顶部栏右侧的搜索框搜索redis。
- 在搜索页面的搜索结果中会出现一条：`redis PHP extension for interfacing with redis`。
- 在点进去的页面可以看到不同版本的redis扩展。
- 根据php版本，下载对应的扩展，windows环境下载dll文件。
- 本人下载下来的是个压缩文件叫php_redis-5.3.4-7.4-ts-vc15-x64。
- 解压之后在里面找到php_redis.dll文件，复制到php扩展目录下。
- 在php.ini中增加一行：`extension="php_redis.dll"`。