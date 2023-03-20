## 常用函数

- 2021-11-08

### http_build_query()

http_build_query()会把布尔值true转换成1。

如果一定要传递true字符串，赋值的时候用true的字符串值，不要直接使用布尔值。

```php
$arr1 = [
  'true_bool' => true,
  'true_string' => 'true'
];
echo http_build_query($arr1);
//true_bool=1&true_string=true
```

