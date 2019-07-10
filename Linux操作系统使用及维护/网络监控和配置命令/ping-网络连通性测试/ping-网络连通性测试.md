# ping 网络连通性测试

在Linux或Windows上，都存在一个`ping`命令，可以用来测试本机和目标IP主机之间的网络是否连通。这篇笔记记录Linux下`ping`命令的用法。

## ping的原理

实际上，`ping`是通过ICMP协议的ECHO和REPLY数据包实现的，如果一台主机无法`ping`通，大部分情况是网络不通，也有可能是该主机拒绝发ICMP REPLY包，以防自己被探测到。

## 测试域名或IP

`ping`命令后面可以加上域名或主机IP，如果接的是域名，会自动通过DNS协议转换成一个可用的IP。

```
ubuntu@ubuntu:~$ ping www.bing.com
PING cn-0001.cn-msedge.net (202.89.233.100) 56(84) bytes of data.
64 bytes from 202.89.233.100: icmp_seq=1 ttl=113 time=82.9 ms
64 bytes from 202.89.233.100: icmp_seq=2 ttl=113 time=60.1 ms
64 bytes from 202.89.233.100: icmp_seq=3 ttl=113 time=59.6 ms
64 bytes from 202.89.233.100: icmp_seq=4 ttl=113 time=65.8 ms
^C
--- cn-0001.cn-msedge.net ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 59.640/67.157/82.991/9.470 ms
```

## 常用参数

命令行中使用`ping`命令一般不会加什么参数，但有时我们需要在`shell`或其它脚本中对`ping`进行简单的封装，这种情况下可能用到一些参数调整发送数据包的个数、间隔等参数。

* `-c`：指定测试数据包发送次数，Linux下如果不指定次数会一直`ping`，按`CTRL+C`终止
* `-i`：指定发送数据包的间隔秒数
