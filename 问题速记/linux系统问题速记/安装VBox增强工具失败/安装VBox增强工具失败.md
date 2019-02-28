# 问题

虚拟机内安装virtualbox增强工具失败，服务不能启动，无法自动适应分辨率。使用的是debian testing，内核比较新（4.9）。

# 解决

安装最新virtualbox即可。如果还是有问题，手动安装以下软件包：
```
apt-get install linux-headers-$(uname -a)
apt-get install dkms
```

# 原因

最新内核旧版虚拟机不识别。
