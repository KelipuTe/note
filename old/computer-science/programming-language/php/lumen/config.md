## 配置相关

- 2021-07-07

### 使用laravel的配置方式

想在lumen中如果想使用laravel的配置方式，可以把lumen源代码中laravel的配置文件复制到config目录下。

配置文件可以在`vendor\laravel\lumen-framework\config`目录下找到。

例如：如果在lumen中想同时连接两个数据库。

这个时候需要把vendor的config目录下的database.php配置文件，复制到config目录下。使用方式和在laravel中的使用方式是一样的。

注意，在laravel中，我们可以直接使用config方法读取配置在config目录下的自定义配置文件。

但是在lumen中，在新建config目录并创建配置文件（如：config\message.php）后，需要在bootstrap/app.php中手动加载：`$app->configure('message')`。
