## easyswoole配置文件

- 文档更新时间：2021-07-07
- "easyswoole/easyswoole":"3.4.2"
- "vlucas/phpdotenv":"v5.3.0"

### 框架配置文件

easyswoole框架提供了基础的dev.php和produce.php两个配置文件，根据框架启动命令的不同，只会加载其中一个，所以如果是线上环境和开发环境一样的配置就会要复制一遍，不是太方便的。而且这两个配置文件在框架进入EasySwooleEvent.php文件的initialize()方法之前就加载了，而不修改源码的情况下，initialize()方法是开发者可以进行操作的第一个方法。如果想要修改配置文件的加载逻辑，是无法在不修改框架源码的情况下修改的。

另外一点，像账号或者密码这样的敏感数据也不太适合直接放在代码里提交到代码库里去。所以一个合理的解决方案是额外引入一个配置文件放置敏感信息或者使用数据库加缓存的模式，对于不是特别敏感的配置项就放到常量里去维护。

框架说明文档里特别提到dev.php和produce.php这两个配置文件是不能热重启的。只有onWorkerStart事件之后加载的文件，才能被`> php easyswoole server reload`命令热重启。

### .env文件

使用.env文件也是个不错的选择。首先使用composer命令安装依赖：`$ composer require vlucas/phpdotenv`，然后创建.env文件编写配置参数，然后把.env文件加载到程序里去就行了。之后就可以通过`$_ENV`全局变量使用这些参数了。

```php
$dotenv = \Dotenv\Dotenv::createImmutable(EASYSWOOLE_ROOT, '.env');
$dotenv->load();
```

值得注意的一点是，配置文件中如果有数据库密码这种全局通用信息的，一定要在所有的业务代码用到配置信息之前，把配置信息加载到程序里去。所以比较合理的位置是EasySwooleEvent.php文件的initialize()方法。