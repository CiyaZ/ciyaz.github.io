# hostname主机名

我们登陆到Linux终端后，命令提示符会显示例如`tom@ubuntu:~$`，`tom`很显然是用户名，那么后面的`ubuntu`是什么呢？其实是系统的`hostname`。我们的主机接入网络中，登入不同的主机时，肯定得需要个名字，从而知道自己登入的是哪台机器，`hostname`的作用就在此，没什么特别的意义，仅仅是当前主机名。

## 修改hostname

Ubuntu系统下，可以通过修改`/etc/hostname`文件来修改主机名，修改该文件后，需要重启系统才能生效。

我们也可以使用`hostname`命令，指定一个当前立即生效的主机名（但该命令的修改不是永久性的，重启系统后会恢复）：

例如：
```
hostname ciyaz.com
```

通常，我们是两种方法同时使用的，以便于修改主机名且免去重启系统的麻烦。

## hostname的作用

`hostname`的作用是给当前主机起一个名字，那么在用到这个名字的时候，就能体现出它的作用了。比如使用`sendmail`发邮件时，邮件就会读取这个`hostname`作为邮件服务器的名字。

## hostname和hosts关系

`hostname`和`hosts`是比较容易混淆的概念，其实两者没有任何关系。

`hosts`用于域名解析，通常我们将当前主机名解析到`127.0.0.1`。在早期的网络中，没有DNS服务器，解析域名全部通过`hosts`文件来实现。后来随着域名越来越多，`hosts`文件不够用，DNS服务和协议才被设计出来。
