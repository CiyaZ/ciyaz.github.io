# artisan-命令行

`artisan`是Laravel框架的命令行接口，我们开发过程中用到的许多框架提供的功能，都是通过它实现的。`artisan`实际上是一个可以用解释器执行的PHP文件，因此使用时应该使用类似`php artisan xxx`的形式。

除此之外，我们也可以自定义命令实现。

## artisan命令行

查阅所有可用命令：

```
php artisan list
```

查阅命令帮助：

```
php artisan help <命令>
```

具体的命令和功能在相关章节都有介绍，这里就不重复赘述了。

## tinker交互式解释器

Laravel中，`tinker`是一个基于`PsySH`的交互式命令行解释器，类似Django的`shell`命令，可以用来交互式的执行PHP脚本，以及操作ORM等功能。

```
php artisan tinker
```
