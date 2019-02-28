# redis扩展

PHP连接Redis需要单独装扩展库，本篇笔记记录如何安装和使用PHP的redis扩展。

## 在Windows下安装Redis扩展

在Windows开发环境下，用PHP连接redis需要做如下步骤：

1. 本地安装好PHP等开发环境，以及redis服务器
2. 安装扩展`php_redis`和`php_igbinary`
3. 配置`php.ini`加载这两个扩展

在Windows下手动安装其实很麻烦，我们首先要明确自己PHP的版本号、`ts`还是`nts`版、编译使用的MSVC的版本，然后去下面这两个地址分别下载两个扩展：

1. [php_igbinary](https://windows.php.net/downloads/pecl/releases/igbinary/)
2. [php_redis](https://windows.php.net/downloads/pecl/releases/redis/)

下载好后，解压出里面的`dll`文件，放进PHP安装目录的`ext`文件夹中。

编辑`php.ini`，加入`extension`配置：

```
extension=php_igbinary.dll
extension=php_redis.dll
```

我们可以用代码尝试连接：
```php
$redis = new redis();
$redis->connect('127.0.0.1', '6379') || die("连接失败！");
```
