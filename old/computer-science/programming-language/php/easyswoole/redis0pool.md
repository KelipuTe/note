## easyswoole使用redis连接池

- 文档更新时间：2021-07-07
- "easyswoole/easyswoole":"3.4.2"
- "easyswoole/redis":"1.3.7"
- "easyswoole/redis-pool":"2.2.1"

### 使用redis-pool组件

redis-pool是基于pool通用连接池和redis协程客户端封装的组件。

```shell
> composer require easyswoole/pool
> composer require easyswoole/redis
> composer require easyswoole/redis-pool
```

不同的连接池可以通过名字（`$poolName`参数）加以区分，如果没有名字的话会使用默认的名字。

在使用之前还需要注册连接池，由于需要在所有的业务使用之前就注册好，所以比较合适的地方是EasySwooleEvent.php文件的initialize()方法。

```php
$configData = [
  'host'      => '127.0.0.1',
  'port'      => '6379',
  'auth'      => 'password',
  'db'        => 0,
  'serialize' => 0, 
]
//创建redis配置
$redisConfig = new \EasySwoole\Redis\Config\RedisConfig($configData);
//注册redis连接池
$redisPool = \EasySwoole\RedisPool\RedisPool::getInstance()
  ->register($redisConfig, $poolName);
//设置连接池参数
$redisPool->setMinObjectNum($minNum);
$redisPool->setMaxObjectNum($maxNum);
```

在用完之后一定要归还（释放）redis连接对象，否则连接池资源耗尽后，后续程序就无法继续获得redis连接对象了。

```php
//获取连接池对象
$redisPool = \EasySwoole\RedisPool\Redis::getInstance()->get($poolName);
//获取redis连接对象
$redisObj = $redisPool->getObj();
//sql操作
$redisObj->set('test-redis-pool','test redis pool data');
$dbData = $redisObj->get('test-redis-pool');
//释放redis连接对象
$redisPool->recycleObj($redisObj);
```
