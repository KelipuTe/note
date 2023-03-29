## 请求相关

#### 获取url

如果控制器方法接收Illuminate\Http\Request，可以直接使用url方法。

```php
public function test(Request $req) 
{
  $url = $req->url();
}
```

如果控制器方法不接收Illuminate\Http\Request，可以使用下面三种方法。

```php
$url = Request::getRequestUri();
$url = URL::current();
$url = Input::url();
```

### 操作请求的原始数据

使用Request::all方法可以获取到所有的请求参数构成的数组。如果想要请求中的一部分参数，则可以使用Request::only方法获取指定的参数构成的数组。或者使用Request::except方法移除不需要的参数，返回由剩余的参数构成的数组。

但是Request::only和Request::except两个方法并不会对原始数据进行修改。如果需要在中间件层，过滤掉无用的参数，或者追加新的参数，就需要对请求的原始数据进行修改。

这时就需要使用 Request::offsetSet和Request::offsetUnset两个方法。Request::offsetSet和Request::offsetUnset两个方法都调用了Request::getInputSource方法得到请求的原始参数。

```php
protected function getInputSource()
  {
    if ($this->isJson()) {
      return $this->json();
    }
    return in_array($this->getRealMethod(), ['GET', 'HEAD']) ? $this->query : $this->request;
  }
```

这里请求的原始数据又分了三个来源GET请求、非GET请求和Json请求。

Illuminate\Http\Request继承了Symfony\Component\HttpFoundation\Request。上面代码片段中的`$this->query`和`$this->request`都可以直接找到。都是 Symfony\Component\HttpFoundation\ParameterBag的实例。Json请求的参数则相对复杂。

上面代码片段中的`$this->query`和`$this->request`都可以在Illuminate\Http\Request中直接使用。如果在控制器里，则可以通过`$request->query`和`$request->request`直接调用。

另外，在给laravel项目发json请求的时候要带上header，要不然可能会解析错误。
