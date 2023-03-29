## 常用命令

### 安装指定版本的扩展

```
#安装5.5版本的xxx扩展
composer require xxx ^5.5
#安装dev-master版本的xxx扩展
composer require xxx:dev-master
```

### 删除扩展

```
#移除一个
composer remove xxx
#移除多个
composer remove xxx1 xxx2
```

移除扩展只是从composer.json文件中移除了扩展的依赖。扩展移除之后，扩展的代码还是在vender目录里的，只是不会被composer自动加载了。