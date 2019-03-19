# vsftpd FTP文件服务

vsftpd是Linux下几乎必装的FTP服务端软件，这篇笔记简单记录一下vsftpd的安装和配置。

## 安装vsftpd

```
sudo apt-get install vsftpd
```

安装好后，其实就可以使用了。默认情况下，我们可以用我们登入系统的用户名和密码登入FTP服务，默认的FTP位置为用户的家目录。

## 配置vsftpd

vsftpd的配置文件在`/etc/vsftpd.conf`，我们可以打开观察一下，注释都非常详细。这里主要介绍几个重要的配置：

是否允许匿名访问，如果我们配置的是一个公开下载服务，就可以允许（但一般不会允许写权限）。

```
anonymous_enable=NO
```

目录是否可写。

```
write_enable=YES
```

分配给FTP的目录，默认为家目录，这里我们手动指定一个其它目录。

```
local_root=/mnt/hdd/smb
```
