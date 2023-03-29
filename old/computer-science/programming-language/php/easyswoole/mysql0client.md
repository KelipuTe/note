## easyswoole使用mysql客户端

- 文档更新时间：2021-07-06
- "easyswoole/easyswoole":"3.4.2"
- "easyswoole/mysqli":"2.2.8"

### 构造客户端

easyswoole没有提供现成的mysql客户端，所以这里自行实现一个。需要框架提供的mysqli客户端组件。

```shell
> composer require easyswoole/mysqli
```

客户端构造函数接受一个`\EasySwoole\Mysqli\Config`实例。这里的配置参数从外部传入，便于连接不同的mysql数据库。

```php
use EasySwoole\Mysqli\Client;
use EasySwoole\Pool\ObjectInterface;

class MysqliClient extends Client implements ObjectInterface
{
  public function __construct(\EasySwoole\Mysqli\Config $config)
  {
    parent::__construct($config);
  }

  function gc()
  {
    $this->close();
  }

  function objectRestore()
  {
  }

  function beforeUse(): ?bool
  {
    return true;
  }
}
```

### 使用客户端

直接创建一个客户端实例出来就可以了。不过使用完之后记得要使用客户端的gc()方法回收资源。

```php
$configData = [
  'host'     => '127.0.0.1',
  'port'     => '3306',
  'user'     => 'admin',
  'password' => 'secret',
  'database' => 'dbname',
  'charset'  => 'utf8mb4'
];
//设置配置参数并创建一个客户端对象
new MysqliClient(new \EasySwoole\Mysqli\Config($configData));
```

