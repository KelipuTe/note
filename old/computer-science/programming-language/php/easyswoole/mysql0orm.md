## easyswoole使用orm

- 文档更新时间：2021-07-07
- "easyswoole/easyswoole":"3.4.2"
- "easyswoole/mysqli":"2.2.8"
- "easyswoole/pool":"1.0.13"

### 配置orm

orm模式的使用相对简单一点，不需要像自定义mysql连接池那样额外创建一些东西。需要pool通用连接池、框架提供的mysqli客户端组件和框架提供的orm组件。

```shell
> composer require easyswoole/pool
> composer require easyswoole/mysqli
> composer require easyswoole/orm
```

然后就是在EasySwooleEvent.php文件的initialize()方法里注册相关信息了。mysql的配置参数和客户端的参数格式是一样的。

```php
$configData = [
  'host'     => '127.0.0.1',
  'port'     => '3306',
  'user'     => 'admin',
  'password' => 'secret',
  'database' => 'dbname',
  'charset'  => 'utf8mb4'
];
//设置orm配置参数
$ormConfig = new \EasySwoole\ORM\Db\Config($configData);
$ormConfig->setMinObjectNum(3);
$ormConfig->setMaxObjectNum(5);
//注册orm配置参数
\EasySwoole\ORM\DbManager::getInstance()->addConnection(
  new \EasySwoole\ORM\Db\Connection($ormConfig), $connectionName);
```

这样orm的连接池就配置好了，orm会自行调用，然后是具体的orm使用流程了。另外orm模式框架自身提供了事务操作。

如果有需要访问不同数据库的需求，在构造连接池的时候，设定一个名字就可以了，然后在orm模型中配置connectionName参数指定连接池的名字。默认情况下构造连接池和orm模型的这两个配置的名字都叫default。

#### orm的sql日志

orm的sql日志可以通过orm提供的onQuery事件实现。

```php
\EasySwoole\ORM\DbManager::getInstance()->onQuery(function ($res, $builder, $start) {
    $builder->getLastQuery()
});
```

这里的builder参数就是上文mysql连接池中提到的lastQueryBuilder参数，通过builder的getLastQuery()方法，就可以拿到填充数据之后最终执行的sql语句。