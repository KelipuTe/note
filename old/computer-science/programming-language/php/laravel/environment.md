## 环境相关

- 2021-11-08

### 检测是否是从命令行执行的

方案1：使用laravel框架提供的runningInConsole方法。

```php
if (app()->runningInConsole()) {
    //运行在命令行下
}
```

方案2：可以使用php自身提供的php_sapi_name方法。

```php
if (php_sapi_name() == 'cli') {
    //运行在命令行下
}

if (strpos(php_sapi_name(), 'cli') !== false) {
    //运行在命令行下
}
```