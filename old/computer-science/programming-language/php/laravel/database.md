## 数据库相关

- 2021-11-08
- linux
- laravel8

### 单次获取执行的sql

```php
DB::enableQueryLog(); //开启sql日志
DB::flushQueryLog(); //清空sql日志

//数据库操作

$queryLog = DB::getQueryLog(); //获取sql日志
DB::disableQueryLog(); //关闭sql日志
```

获取到的sql日志长这样，如果执行了多次那数组就会有多个元素。

```php
array(1) {
  [0]=>
  array(3) {
    ["query"]=>
    string(71) "select * from `test` where `id` = ?"
    ["bindings"]=>
    array(1) {
      [0]=>
      int(1)
    }
    ["time"]=>
    float(380.85)
  }
}
```

### 持续监听执行的sql

使用laravel框架DB Facade提供的listen方法。在AppServiceProvider（服务提供者）的boot方法中注册查询监听器。

```php
public function boot()
{
  DB::listen(function ($query) {
    // $query->sql 获取预处理sql语句
    // $query->bindings 获取预处理绑定的参数
    // $query->time 获取sql执行时间
  });
}
```

### 连接不同的数据库

首先配置database.php，增加一个mysql链接。

```php
'mysql2' => [
  'driver'    => 'mysql',
  'host'      => 'host2',
  'database'  => 'database2',
  'username'  => 'user2',
  'password'  => 'password2',
  'charset'   => 'utf8mb4',
  'collation' => 'utf8mb4_unicode_ci',
  'prefix'    => '',
]
```

配置完成后，就可以在程序中使用mysql2链接另外一个数据库了。

在使用数据库迁移时，指定数据表使用的链接。

```php
Schema::connection('mysql2')->create('some_table', function($table) {
  $table->increments('id');
});
```

在使用查询构造器时，指定数据库连接。

```php
DB::connection('mysql2')->select(...);
```

在模型中，指定模型数据库连接。

```php
class UserModel extends Eloquent
{
  protected $connection = 'mysql2';
}
```

模型也可以实例化之后，再指定数据库连接。

```php
$userModel = new UserModel();
$userModel->setConnection('mysql2');
$user = $userModel->find(1);

//on方法也是可以的
$userModel->on('mysql2')->find(1); 
```