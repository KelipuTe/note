## easyswoole全局变量

- 文档更新时间：2021-07-07
- "easyswoole/easyswoole":"3.4.2"
- "easyswoole/http":"1.7.19"

在swoole协程当中`$_GET`，`$_POST`等全局变量是不能安全使用的。原因是协程切换下会带来数据污染问题。easyswoole在spl包中，实现了一个SplContextArray，并在主进程的位置，替换了这些全局变量，这使得这些数据的访问是安全的，并在请求结束后自动清理。

可以通过GlobalParamHook实现`$_GET`，`$_POST`等全局变量的安全使用。注意：该特性需要1.6版本以上的http组件库`"easyswoole/http": "^1.6"`。在EasySwooleEvent.php文件中，通过如下方式注册和初始化全局变量。这样注册完成之后就可以在任意地方使用这些全局变量了。

```php
public static function mainServerCreate(EventRegister $register)
{
  GlobalParamHook::getInstance()->hookDefault();
}

public static function onRequest(Request $request, Response $response): bool
{
  GlobalParamHook::getInstance()->onRequest($request, $response);
}
```

这里需要注意，上面面说的任意地方不包括协程内部。如果碰到需要在协程中使用`$_GET`，`$_POST`中的数据时，一定要在进入协程之前把数据取出来，然后传到协程里去。在协程中这些全局变量中的数据并不存在。
