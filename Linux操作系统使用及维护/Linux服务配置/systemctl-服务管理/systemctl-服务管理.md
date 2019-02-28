# systemctl 服务管理

systemctl是systemd的管理命令之一，可以用于管理systemd系统和服务，用于取代原来的service和chkconfig等命令，在较新的发行版中已经广泛使用。这篇笔记记录systemctl命令的基本使用。

## 服务管理命令

```
# 启动服务
systemctl start service

# 结束服务
systemctl stop service

# 重启服务
systemctl restart service

# 查看服务状态
systemctl status service

# 开机时启用服务
systemctl enable service

# 开机时禁用服务
systemctl disable service

# 查看所有可管理资源状态
systemctl list-unit-files
```

注意：enable和disable可以控制服务是否开机启动。

## systemd系统管理

```
# 查看systemd系统版本
systemctl --version

# 重启系统
systemctl reboot

# 系统关机
systemctl poweroff
```

除了重启和关机，还可以使用如下指令：

* halt CPU挂起
* suspend 系统暂停
* hibernate 系统休眠

不过在虚拟机中测试，除了重启，无论如何也没法唤醒系统。
