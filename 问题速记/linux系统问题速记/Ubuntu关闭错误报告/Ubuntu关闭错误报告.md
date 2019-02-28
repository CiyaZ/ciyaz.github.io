# Ubuntu 关闭错误报告

Ubuntu有一个错误报告功能，应用程序非正常停止时，就会弹出一个窗口，询问用户是否进行“错误报告”。这个功能对于普通用户来说倒也没什么，但是对于我们开发者来说真的很烦，我们可以通过配置文件把它关掉。

配置文件位置
```
/etc/default/apport
```

改成如下内容即可：
```
# set this to 0 to disable apport, or to 1 to enable it
# you can temporarily override this with
# sudo service apport start force_start=1
enabled=0
```

这样这个服务就不会运行了，如果想再次打开，`enabled`置位`1`即可。
