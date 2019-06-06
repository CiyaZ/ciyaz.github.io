# Laravel开发环境搭建

Laravel是一个时下非常流行的PHP开发框架，这个框架基于composer进行构建，设计吸取了ROR等框架的优点，在许多开发人员中评价都不错。当然，它的缺点也相对明显，Laravel的性能奇差，另一方面，它设计的实在是复杂，很多人认为Laravel这种重型框架失去了PHP这门语言的初衷。

无论如何，Laravel现已得到了广泛的使用，并且更新速度非常之快，这里我们就简单学习体验一下Laravel。

Laravel官网：[https://laravel.com/](https://laravel.com/)

## 环境搭建

Laravel环境搭建非常复杂，而且坑比较多，PHP本身环境就比较难搞，而且composer也基本是我见过的最差的包管理器（LOGO还特别丑），因此这里也是尝试了半天才将初始工程运行起来。

### Windows开发环境搭建

Laravel官网对PHP扩展的说明含糊不清，而且依赖的扩展随着Laravel版本升级可能还会变，我们按照官网工程搭建一步步来，遇到错误再找对应的办法就行了。

我这里使用的是PHP7.1.29，开启了以下扩展：

```ini
extension=php_curl.dll
extension=php_fileinfo.dll
extension=php_mbstring.dll
extension=php_mysqli.dll
extension=php_openssl.dll
extension=php_pdo_mysql.dll
```

安装Laravel命令行工具：
```
composer global require laravel/installer
```

创建项目：
```
laravel new demo01
```

用PHP内置服务器启动项目：
```
php artisan serve
```

注：`artisan`是Laravel中一个PHP命令行工具。

### Linux开发环境搭建

（待补充）
