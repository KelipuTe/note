## easyswoole使用mysql连接池

- 文档更新时间：2021-07-06
- "easyswoole/easyswoole":"3.4.2"
- "easyswoole/mysqli":"2.2.8"
- "easyswoole/pool":"1.0.13"

### 构造连接池

easyswoole没有提供现成的mysql连接池，所以这里自行实现一个。需要pool通用连接池和框架提供的mysqli客户端组件。

```shell
> composer require easyswoole/pool
> composer require easyswoole/mysqli
```

在另一篇文档中，我们实现了一个mysql客户端，这里沿用那个mysql客户端，然后基于pool通用连接池实现一个mysql连接池。连接池的工作就是创建n个客户端并负责维护它们。

连接池构造函数接受一个`\EasySwoole\Pool\Config`实例和mysql客户端配置参数。

```php
use EasySwoole\Pool\AbstractPool;

class MysqliPool extends AbstractPool
{
  protected $configKey;

  protected $configData;

  public function __construct(\EasySwoole\Pool\Config $config, array $configData)
  {
    $this->configData = $configData;
    parent::__construct($config);
  }

  protected function createObject()
  {
    //创建mysqli配置类
    $mysqliConfig = new \EasySwoole\Mysqli\Config($this->configData);
    //创建一个mysqli客户端
    return new MysqliClient($mysqliConfig);
  }
}
```

### 注册连接池

不同的连接池可以通过名字（`$poolName`参数）加以区分，如果没有名字的话会使用默认的名字。

在使用之前还需要注册连接池，由于需要在所有的业务使用之前就注册好，所以比较合适的地方是EasySwooleEvent.php文件的initialize()方法。

```php
$configData = [
  'host'     => '127.0.0.1',
  'port'     => '3306',
  'user'     => 'admin',
  'password' => 'secret',
  'database' => 'dbname',
  'charset'  => 'utf8mb4'
];
//设置连接池参数
$mysqliPoolConfig = new \EasySwoole\Pool\Config();
$mysqliPoolConfig->setMinObjectNum(2);
$mysqliPoolConfig->setMaxObjectNum(10);
//注册mysql连接池
\EasySwoole\Pool\Manager::getInstance()
  ->register(new MysqliPool($mysqliPoolConfig, $configData), $poolName);
```

### 使用连接池

使用的话就很简单了。需要注意的就是，用完之后一定要归还mysqli客户端对象。所以这里建议用try-catch-finally包起来。

```php
//获取连接池对象
$mysqlPool = \EasySwoole\Pool\Manager::getInstance()->get($poolName);
//获取mysqli客户端
$mysqliClient = $mysqlPool->getObj();
try {
  //sql操作
  $mysqliClient->queryBuilder()->get('table_name');
  $dbData = $mysqliClient->execBuilder();
} catch (\Exception $e) {
} finally {
  //释放mysqli客户端
  $mysqlPool->recycleObj($mysqliClient);
}
```

### 事务操作

自主实现的MySQL连接池如果想实现事务操作也需要自行编码，需要用到`\Swoole\Coroutine\MySQL`类中的begin()，commit()，rollback()等事务操作方法。这个类在Client类执行execBuilder()方法的时候（构造完语句最终开始执行的时候调用的方法），通过Client类的connect()方法构造。所以如果想主动控制事务，需要把connect()方法的调用提到外层。这个方法是幂等的，所以不会影响内部的逻辑。

```php
$mysqlPool = \EasySwoole\Pool\Manager::getInstance()->get($poolName);
$mysqliClient = $mysqlPool->getObj();
try {
  $mysqliClient->connect();
  //开启事务
  $mysqliClient->mysqlClient()->begin();
  $mysqliClient->queryBuilder()->get('table_name');
  $dbResult1 = $mysqliClient->execBuilder();
  //提交事务
  $mysqliClient->mysqlClient()->commit();
} catch (\Exception $e) {
} finally {
  $mysqlPool->recycleObj($mysqliClient);
}
```

### sql日志

开发环境可以加一个自动输出sql执行语句的机制，方便排查问题和sql调优。实现方式是自定义mysqli客户端的execBuilder()方法，简单一点，复制一下加点逻辑就行了。

可以使用`\EasySwoole\EasySwoole\Core::getInstance()->isDev()`方法判断是不是开发环境。然后mysqli客户端提供了lastQueryBuilder参数保存最后一次sql执行的各项参数，比如getLastQuery()方法就可以获取填充数据之后最终执行的sql语句。

```php
function execBuilder(float $timeout = null)
{
  $this->lastQueryBuilder = $this->queryBuilder;
  if (ConfigHelper::isDevOrTestENV()) {
    // 开发测试环境输出sql
    LogHelper::echoToFile(
      $this->lastQueryBuilder->getLastQuery(), 'client_sql', 'sql log');
  }
  $start = microtime(true);
  if ($timeout === null) {
    $timeout = $this->config->getTimeout();
  }
  try {
    $this->connect();
    $stmt = $this->mysqlClient()
      ->prepare($this->queryBuilder()->getLastPrepareQuery(), $timeout);
    $ret = null;
    if ($stmt) {
      $ret = $stmt->execute($this->queryBuilder()->getLastBindParams(), $timeout);
    } else {
      $ret = false;
    }
    if ($this->onQuery) {
      call_user_func($this->onQuery, $ret, $this, $start);
    }
    if ($ret === false && $this->mysqlClient()->errno) {
      throw new Exception($this->mysqlClient()->error);
    }
    return $ret;
  } catch (\Throwable $exception) {
    throw $exception;
  } finally {
    $this->reset();
  }
}
```
