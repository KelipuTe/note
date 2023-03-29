## php使用mongodb

- 2020-12-13
- "mongodb/mongodb":"1.6.1"

| 参考来源                                           |
| -------------------------------------------------- |
| [PHP手册](https://www.php.net/manual/zh/index.php) |
| [MongoDB官方文档](https://www.mongodb.com/)        |

首先需要注意这篇文档使用的php版本是php7，如果是别的版本的php不一定适用。

### 安装mongodb扩展

mongodb的扩展是需要额外安装的。

如果觉得原生的扩展不好用，可以使用mongodb官方提供的composer扩展包。、

关于扩展包的文档可以在mongodb官方网站的[Drivers](https://docs.mongodb.com/drivers/)标签下找到，Drivers目录下面有官方提供的不同语言的扩展包。

在安装完php的mongodb扩展之后，就可以直接通过composer命令`composer require mongodb/mongodb`安装了。

这里需要注意一下php的mongodb扩展的版本和composer扩展包版本的依赖关系。

#### 用pecl安装mongodb扩展

使用pecl来安装：`pecl install mongodb`。

安装完成后在php.ini配置文件里添加mongodb扩展`extension=mongodb.so`。

保存后使用`php -m`命令或者`php -i`命令检查一下扩展是否启用了。

#### 编译安装mongodb扩展

首先找到php-config目录的位置，这个样例用的是swoole官方提供的docker镜像的alpine linux环境。

在镜像phpswoole/swoole:4.4.23-php7.4中，php-config目录在`/usr/local/bin`目录下。

这里用mongodb扩展举例，使用命令下载mongodb扩展的tgz压缩包。

也可以去pecl（[https://pecl.php.net/](https://pecl.php.net/)）上先下载好，再手动传到linux环境。

```shell
#解压
> tar zxfv mongodb-1.9.1.tgz
#进入解压后的目录
> cd mongodb-1.9.1
#生成configure配置文件
> phpize
#配置
> ./configure --with-php-config=/usr/local/bin/php-config
```

编译完成会看见下面的输出。

```shell
#编译
> make
...
----------------------------------------------------------------------
Libraries have been installed in:
   /code/mongodb-1.9.1/modules

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,--rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------

Build complete.
```

安装完成会看见下面的输出，生成的so文件会放到下面这个目录里。

```shell
#安装
> make install
Installing shared extensions:     /usr/local/lib/php/extensions/no-debug-non-zts-20190902/
```

安装完成后在php-ini文件里加上mongodb的扩展mongodb.so。

使用`php --ini`命令找到配置文件。

```shell
> php --ini
Configuration File (php.ini) Path: /usr/local/etc/php
Loaded Configuration File:         /usr/local/etc/php/php.ini
Scan for additional .ini files in: /usr/local/etc/php/conf.d
Additional .ini files parsed:      /usr/local/etc/php/conf.d/docker-php-ext-sockets.ini,
/usr/local/etc/php/conf.d/docker-php-ext-sodium.ini,
/usr/local/etc/php/conf.d/docker-php-ext-swoole.ini
```

如果`Loaded Configuration File`这项是空的，就去`Configuration File (php.ini) Path`创建一个新的php.ini文件。通常这个目录下有php.ini-development这样的样例文件，复制一个就行了。

### 相关类名

| 类名 | 用途 |
| --- | --- |
| MongoDB\Driver\Manager| 连接数据库 |
| MongoDB\BSON\ObjectId | 生成`_id` |
| MongoDB\Driver\BulkWrite | 插入，修改，删除 |
| MongoDB\Driver\WriteConcern | 配置 |
| MongoDB\Driver\WriteResult | 插入，修改，删除的结果 |
| MongoDB\Driver\Query | 查询 |
| MongoDB\Driver\Cursor | 查询的结果 |

### 连接

```php
$manager = new Manager('mongodb://user:secret@localhost:27017');
```

这样就创建了一个连接，创建连接涉及到Manager类。

在Manager类的构造函数中，可以传入一个mongodb的连接url。

数据库访问地址是localhost，端口是27017，用户名是user，用户密码是secret。

### 插入

插入需要用到BulkWrite，WriteConcern，WriteResult类。

BulkWrite类用于处理写入的信息，可以同时插入一条或者多条记录。WriteResult类里记录了插入操作处理的结果。

进行插入操作的时候使用的是BulkWrite类的insert方法。

#### 单条插入

```php
$insertField = ['_id' => new ObjectId(), 'name' => '111'];
$bulkWrite = new BulkWrite();
$insertId = $bulkWrite->insert($insertField);

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite('db_name.collection_name', $bulk, $writeConcern);
```

BulkWrite类的insert()方法会处理要插入的数据。

如果插入的数据没有`_id`字段的话就会生成一个，这个方法返回一个ObjectId对象，包含`_id`字段的信息。

var_dump()出来就是下面这样的数据，可以使用`__toString()`方法转换成文本。

关于`_id`字段，详细的可以看mongodb的相关信息。

```php
object(MongoDB\BSON\ObjectId)#124 (1) {
  ["oid"]=>
  string(24) "5fd6fda1df5c274bc51e1751"
}
```

BulkWrite类处理完成的数据交给Manager类的executeBulkWrite()方法处理。

executeBulkWrite()方法的第一个参数是一个字符串`'db_name.collection_name'`，db_name表示数据库名，collection_name表示集合名。

executeBulkWrite()方法返回一个WriteResult对象。里面记录本次操作的信息，比如这里nInserted=1就表示刚才插入了一条数据。

```php
object(MongoDB\Driver\WriteResult)#133 (9) {
  ["nInserted"]=>
  int(1)
  ["nMatched"]=>
  int(0)
  ["nModified"]=>
  int(0)
  ["nRemoved"]=>
  int(0)
  ["nUpserted"]=>
  int(0)
  ["upsertedIds"]=>
  array(0) {
  }
  ["writeErrors"]=>
  array(0) {
  }
  ["writeConcernError"]=>
  NULL
  ["writeConcern"]=>
  object(MongoDB\Driver\WriteConcern)#127 (2) {
    ["w"]=>
    string(8) "majority"
    ["wtimeout"]=>
    int(1000)
  }
}
```

#### 多条插入

```php
$insertFieldBulk = [
    ['_id' => new ObjectId(), 'name' => '222', 'age' => 20, 'status' => 1],
    ['_id' => new ObjectId(), 'name' => '333', 'age' => 30, 'status' => 1],
    ['_id' => new ObjectId(), 'name' => '444', 'age' => 40, 'status' => 0],
    ['_id' => new ObjectId(), 'name' => '555', 'age' => 40, 'status' => 0],
];

$bulkWrite = new BulkWrite();
$insertIdList = [];
foreach ($insertFieldBulk as $index => $itemInsertField) {
    $insertIdList[$index] = $bulkWrite->insert($itemInsertField);
}

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite($this->getCollectionName(), $bulkWrite, $writeConcern);
```

`$insertIdList`和​`$writeResult`的输出如下，这里插入了4条数据。

```php
//$insertIdList
array(4) {
  [0]=>
  object(MongoDB\BSON\ObjectId)#133 (1) {
    ["oid"]=>
    string(24) "5fd705e3d6f7886f8d7bb741"
  }
  [1]=>
  object(MongoDB\BSON\ObjectId)#134 (1) {
    ["oid"]=>
    string(24) "5fd705e3d6f7886f8d7bb742"
  }
  [2]=>
  object(MongoDB\BSON\ObjectId)#135 (1) {
    ["oid"]=>
    string(24) "5fd705e3d6f7886f8d7bb743"
  }
  [3]=>
  object(MongoDB\BSON\ObjectId)#136 (1) {
    ["oid"]=>
    string(24) "5fd705e3d6f7886f8d7bb744"
  }
}
//$writeResult
object(MongoDB\Driver\WriteResult)#139 (9) {
  ["nInserted"]=>
  int(4)
  ["nMatched"]=>
  int(0)
  ["nModified"]=>
  int(0)
  ["nRemoved"]=>
  int(0)
  ["nUpserted"]=>
  int(0)
  ["upsertedIds"]=>
  array(0) {
  }
  ["writeErrors"]=>
  array(0) {
  }
  ["writeConcernError"]=>
  NULL
  ["writeConcern"]=>
  object(MongoDB\Driver\WriteConcern)#131 (2) {
    ["w"]=>
    string(8) "majority"
    ["wtimeout"]=>
    int(1000)
  }
}
```

### 查询

查询需要用到Query，Cursor类。Query类用于处理查询条件，Cursor类用于记录查询的结果。

Query类的`$filter`参数用于设置查询的筛选条件，`$options`参数可以设置limit，skip，sort等条件。

关于查询语句的写法可以参考mongodb官方提供的[查询文档](https://docs.mongodb.com/manual/tutorial/query-documents/)，还有[操作符文档](https://docs.mongodb.com/manual/reference/operator/query/#query-selectors)。

当查询条件为空时，就会查询全表。

#### 通过id查询

```php
$objectId = '5fd6fda1df5c274bc51e1751';
$filter = ['_id' => new ObjectId($objectId)];
$options = [];
$query = new Query($filter, $options);

$manager = new Manager($this->uri);
$cursor = $manager->executeQuery($this->getCollectionName(), $query);
```

Manager的executeQuery()方法执行查询并返回一个Cursor对象。可以调用Cursor类的toArray()方法，将Cursor对象转换成数组，数组中就是查询的结果，结果以对象的形式呈现。

```php
array(1) {
  [0]=>
  object(stdClass)#133 (4) {
    ["_id"]=>
    object(MongoDB\BSON\ObjectId)#132 (1) {
      ["oid"]=>
      string(24) "5fd6fda1df5c274bc51e1751"
    }
    ["name"]=>
    string(3) "222"
    ["age"]=>
    int(10)
    ["status"]=>
    int(1)
  }
}
```

#### 通过条件查询

单条件和操作符：

```php
// 查询name=222的记录
$filter = ['name' => '222'];
// 查询age大于20的记录，$gt表示大于
// 类似的，$lt表示小于，$gte表示大于等于
$filter = ['age' => ['$gt' => 20]];
```

多条件之间的与和或的关系：

```php
//age>20 and status=0
$filter = [
  'age'    => ['$gt' => 20],
  'status' => 0
];
//age>20 or status=0
$filter = [
  '$or' => [
    'age'    => ['$gt' => 20],
    'status' => 0
  ]
];
//name='222' and (age>20 or status=0)
$filter = [
  'name' => '222'
  '$or' => [
    'age'    => ['$gt' => 20],
    'status' => 0
  ]
];
```

### 更新

更新需要用到BulkWrite，WriteConcern，WriteResult类。

BulkWrite类用于处理更新的信息，可以同时修改一条或者多条记录。WriteResult类里记录了更新操作处理的结果。

进行更新操作的时候使用的是BulkWrite的update()方法。

更新这里需要注意一下$options参数的multi和upsert参数。

multi=false时只会更新符合条件的第一条记录，multi=true时会更新全部符合条件的记录。

upsert=false时找不到匹配的记录不会新建一条记录，upsert=true时找不到匹配的记录会新建一条记录。

更新同时涉及查询和写入操作，查询条件的写法和上文中单纯的查询操作的写法是一样的。

但是，写入操作的写法和单纯的插入操作是不一样的，写入操作也是有类似查询条件的操作符一样的东西。

详细的看mongodb官方提供的[更新文档](https://docs.mongodb.com/manual/reference/command/update/#update-with-an-aggregation-pipeline)。

#### 通过id更新

```php
$objectId = '5fd6fda1df5c274bc51e1751';
$bulkWrite = new BulkWrite;
$filter = ['_id' => new ObjectId($objectId)];
$newObj = ['$set' => ['name' => '222222']];
$options = ['multi' => true, 'upsert' => false];
$bulkWrite->update($filter, $newObj, $options);

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite($this->getCollectionName(), $bulkWrite, $writeConcern);
```

`$newObj`参数就是操作数据的参数，样例里的`$set`操作符表示追加或者修改字段，常用的还有`$unset`操作符表示移除字段。

样例操作的结果`$writeResult`如下，nMatched表示匹配到几条数据，nModified表示修改了几条数据。

```php
object(MongoDB\Driver\WriteResult)#132 (9) {
  ["nInserted"]=>
  int(0)
  ["nMatched"]=>
  int(1)
  ["nModified"]=>
  int(1)
  ["nRemoved"]=>
  int(0)
  ["nUpserted"]=>
  int(0)
  ["upsertedIds"]=>
  array(0) {
  }
  ["writeErrors"]=>
  array(0) {
  }
  ["writeConcernError"]=>
  NULL
  ["writeConcern"]=>
  object(MongoDB\Driver\WriteConcern)#129 (2) {
    ["w"]=>
    string(8) "majority"
    ["wtimeout"]=>
    int(1000)
  }
}
```

#### 通过条件更新

```php
$bulkWrite = new BulkWrite;
// update set name='222' where name='111'
$filter = ['name'=>'111'];
$newObj = ['$set' => ['name' => '222']];
// 更新第一条
// $options = ['multi' => false, 'upsert' => false];
// 更新全部
$options = ['multi' => true, 'upsert' => false];
$bulkWrite->update($filter, $newObj, $options);

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite($this->getCollectionName(), $bulkWrite, $writeConcern);
```

### 删除

更新需要用到BulkWrite，WriteConcern，WriteResult类。

BulkWrite类用于处理删除的信息，可以同时删除一条或者多条记录。WriteResult类里记录了删除操作处理的结果。

进行更新操作的时候使用的是BulkWrite的delete()方法。

这里需要注意一下`$options`参数的limit参数。limit=false时会删除全部符合条件的记录，limit=true时只会删除符合条件的第一条记录。

删除操作的查询条件的写法和上文中单纯的查询操作的写法是一样的，但是这里特别要注意，如果删除操作的查询条件时空的，会删除全部的数据。

#### 通过id删除

```php
$objectId = '5fd6fda1df5c274bc51e1751';
$bulkWrite = new BulkWrite;
$filter = ['_id' => new ObjectId($objectId)];
$deleteOptions = ['limit' => false];
$bulkWrite->delete($filter, $deleteOptions);

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite($this->getCollectionName(), $bulkWrite, $writeConcern);
```

#### 通过条件删除

```php
$bulkWrite = new BulkWrite;
$filter = ['name'=>'111'];
$deleteOptions = ['limit' => false];
$bulkWrite->delete($filter, $deleteOptions);

$manager = new Manager($this->uri);
$writeConcern = new WriteConcern(WriteConcern::MAJORITY, 1000);
$writeResult = $manager->executeBulkWrite($this->getCollectionName(), $bulkWrite, $writeConcern);
```

