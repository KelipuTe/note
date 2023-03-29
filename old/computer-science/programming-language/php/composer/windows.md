## windows环境安装和配置composer

- 2021-11-17
- windows-10
- php-7.4
- composer-2.1.12

### 下载

去官方网站（https://getcomposer.org）下载windows环境的安装包。

本人这里下载下来的文件是：Composer-Setup.exe

### 安装

在配置好php之后，再执行composer的安装程序，一路下一步即可。

安装完成后，可以使用`composer -V`命令，查看composer版本。

```
C:\Users\Administrator>composer -V
Composer version 2.1.12 2021-11-09 16:02:04
```

### 切换镜像

这里用的是阿里云的镜像。

```
#全局切换镜像
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
#全局取消镜像配置
composer config -g --unset repos.packagist
#只配置当前项目镜像
composer config repo.packagist composer https://mirrors.aliyun.com/composer/
#取消当前项目镜像配置
composer config --unset repos.packagist
```

比较推荐使用只配置当前项目镜像，因为这个配置可以被带到composer.json里去。即使换了环境，再次执行composer命令时也会使用镜像。