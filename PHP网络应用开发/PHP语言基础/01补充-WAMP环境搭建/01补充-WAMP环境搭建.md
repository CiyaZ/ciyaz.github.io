# WNMP环境搭建

在Linux上开发php固然简单，但是很多入门级开发者并不熟悉Linux，因此在公司的多人项目中，为了大家的开发环境能协调一致，基本都是使用Windows操作系统的。

Windows下有一些WAMP整合包，这里建议不要用，各个生产环境的服务器、PHP、数据库版本都可能各不相同，整合包虽然能直接使用，但是可能造成潜在问题。

这篇笔记记录如何在Windows下搭建开发环境。

## 安装php7

我们去php官网，找到PHP的windows版本，下载线程安全（Thread Safe）版。

* 注1：选择PHP版本时，要注意该版本有对应的XDebug、IDE、各种扩展的支持，最新版本可能存在不支持的情况，学习时也建议下载生产环境用的最多的版本
* 注2：如果使用Apache下载线程安全版，如果使用Nginx下载非线程安全版（该版本不包含Apache的mod支持），这是两种不同的运行机制决定的，Apache使用php模块，Nginx使用`php-fpm`和FastCGI协议。

下载好后，配入`Path`环境变量。

在安装目录，将`php.ini-development`复制一份，重命名成`php.ini`。

## 下载Apache

Apache官方没有提供Windows版本，但是有一些第三方的Windows预编译包，可以在Apache官网找到，我们这里下载的就是Apache2.4版本（Apache似乎停留在这个版本已经很久没更新了）。

在Apache的`conf`文件夹中，找到`httpd.conf`，在文件开头加上这些配置：

```
AddHandler application/x-httpd-php .php
AddType application/x-httpd-php .php .html
LoadModule php7_module "c:/php-ts/php7apache2_4.dll"
PHPIniDir "c:/php-ts"
```

## 安装MySQL

具体请参考`数据库系统/MySQL数据库管理系统/MySQL简介和环境搭建`。

## 启动Apache

双击执行`Apache24/bin/httpd.exe`即可。

服务器启动后，访问`localhost/test.php`，就可以看到php正常工作了。

## 开发IDE配置

和上一篇基本相同，这里就不多做叙述了。

注意：XDebug的默认（也不算默认，就是大家都这么配）9000端口可能和惠普电脑的一大堆服务冲突，换一个就好了。

## 使用Nginx

Nginx的使用在Windows下和Linux基本相同，稍微不同的一个是Windows下Nginx配置文件、可执行文件都放在一起，另外Windows下也没有unix domain socket，因此需要配置到php-fpm的IP和端口，Nginx默认带有PHP的配置，只要取消注释就行了，这里就不多介绍了。
