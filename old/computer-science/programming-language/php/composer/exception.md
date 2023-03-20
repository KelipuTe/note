## 一些异常和解决方案

### 更新composer报错

使用composer时，遇到提示composer版本超过60天，建议更新的提示。

使用`composer self-update`命令更新composer时，遇到下面的报错。

```
$ composer self-update
Updating to version beb64914a37205f6748552d764d9531d99c7f41e (snapshot channel).
   Downloading (100%)
Failed to decode response: zlib_decode(): data error
Retrying with degraded mode, check https://getcomposer.org/doc/articles/troubleshooting.md#degraded-mode for more info
Downloading (100%)

  [ErrorException]
  zlib_decode(): data error

self-update [-r|--rollback] [--clean-backups] [--no-progress] [--update-keys] [--stable] [--preview] [--snapshot] [--set-channel-only] [--] [<version>]
```

这种格式的报错，比较玄学的解决方法是把全局设置的镜像路由里`https`改成`http`。

### 安装ip2region包时报错

```
$ composer require zoujingli/ip2region
Using version ^1.0 for zoujingli/ip2region
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
PHP Fatal error:  Allowed memory size of 1610612736 bytes exhausted (tried to allocate 4096 bytes) in phar://C:/ProgramData/ComposerSetup/bin/composer.phar/src/Composer/DependencyResolver/RuleSetGenerator.php on line 129

Fatal error: Allowed memory size of 1610612736 bytes exhausted (tried to allocate 4096 bytes) in phar://C:/ProgramData/ComposerSetup/bin/composer.phar/src/Composer/DependencyResolver/RuleSetGenerator.php on line 129

Check https://getcomposer.org/doc/articles/troubleshooting.md#memory-limit-errors for more info on how to handle out of memory errors.
E:\GongZuoQu\innpms-server>composer require zoujingli/ip2region
Using version ^1.0 for zoujingli/ip2region
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
PHP Fatal error:  Allowed memory size of 1610612736 bytes exhausted (tried to allocate 4096 bytes) in phar://C:/ProgramData/ComposerSetup/bin/composer.phar/src/Composer/DependencyResolver/RuleSetGenerator.php on line 129

Fatal error: Allowed memory size of 1610612736 bytes exhausted (tried to allocate 4096 bytes) in phar://C:/ProgramData/ComposerSetup/bin/composer.phar/src/Composer/DependencyResolver/RuleSetGenerator.php on line 129

Check https://getcomposer.org/doc/articles/troubleshooting.md#memory-limit-errors for more info on how to handle out of memory errors.
```

这只因为安装包的时候PHP内存溢出了。下面三个方案任选其一即可。

- 修改php.ini文件中的内存最大值配置。
- 使用`COMPOSER_MEMORY_LIMIT=-1 composer require ***`命令去掉内存限制。
- 使用`php -d memory_limit=-1 which composer require ***`命令去掉内存限制。