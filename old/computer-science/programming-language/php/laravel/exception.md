## 一些异常和解决方案

- 2021-11-08
- linux
- laravel8

## 对查询构造器的结果使用toArray方法，报空指针错误

查询构造器的结果有两种获取方式：

第一种情况是使用get方法获取结果，即`Model::where()->get();`。这个时候如果查询结果为空，会返回一个空的查询结果集。

这个时候是可以使用toArray方法的，会得到一个空数组。而且这种情况使用empty判空会返回false，因为空的查询结果集是个实例对象。

另一种情况是使用first方法获取第一条结果，即`Model::where()->first();`。first方法会从查询结果集中取第一条数据，所以从空的查询结果集会取出null。

这个时候使用toArray方法就会报空指针错误。这种情况使用empty判空会返回true。

```php
empty(Model::where()->get()); //false
count(Model::where()->get()); //0
empty(Model::where()->first()); //true
count(Model::where()->first()); //0
```

## 使用chunk方法时，数据没有全部被处理

例如，我们要删除10000条用户数据中前1000条，然后分块执行。

```php
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
  foreach ($users as $user) {
    //通过id删除用户
  }
});
```

这个方法执行过之后会发现，被删除的数据并不是前1000条。

原因是：第一次chunk，取出所有的数据排序并取出第1页，即id=1~100的数据，进行删除操作。第二次chunk，理论上应该取出id=101~200的数据。但是现在 id=1~100已经被删除了，chunk取出的是逻辑上的第2页，也就是现在数据库中id=201~300的数据。这样就导致id=101-200的数据没有被删除。

## 使用gd库创建图片时，返回false

异常场景：使用gd库创建图片时，返回false。

异常原因：图片目录不存在。因为laravel提供的storage方法不会自动创建目录。
