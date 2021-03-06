# traceroute 数据包路径探测

我们从自己的电脑上访问一个网站，实际上我们的数据包会在很多节点之间路由，会经过许许多多的网关，最终抵达目标。`traceroute`工具能够很方便的探测我们的数据包究竟通过了哪些网关节点。

## 基本用法

这里使用的UbuntuServer16.04没有预装`traceroute`，需要先安装：
```
sudo apt-get install traceroute
```

探测一个网络主机：
```
ubuntu@ubuntu:~$ traceroute www.bing.com
traceroute to www.bing.com (202.89.233.100), 30 hops max, 60 byte packets
 1  bogon (192.168.43.221)  3.623 ms  3.401 ms  3.660 ms
 2  * * *
 3  bogon (10.245.101.178)  40.453 ms  40.724 ms  40.620 ms
 4  * * *
 5  113.5.0.21 (113.5.0.21)  41.613 ms  40.748 ms  40.859 ms
 6  113.5.0.9 (113.5.0.9)  40.249 ms  37.072 ms  46.385 ms
 7  221.212.35.245 (221.212.35.245)  46.776 ms 1.190.174.85 (1.190.174.85)  38.758 ms 221.212.35.241 (221.212.35.241)  38.694 ms
 8  219.158.100.13 (219.158.100.13)  53.229 ms  53.348 ms 219.158.14.13 (219.158.14.13)  59.793 ms
 9  124.65.194.90 (124.65.194.90)  59.696 ms 123.126.0.126 (123.126.0.126)  59.578 ms 125.33.186.18 (125.33.186.18)  59.031 ms
10  123.126.8.126 (123.126.8.126)  64.184 ms 124.65.56.178 (124.65.56.178)  64.032 ms 123.126.7.142 (123.126.7.142)  63.965 ms
11  * * *
12  61.148.60.134 (61.148.60.134)  71.952 ms  63.583 ms  66.922 ms
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * * *
27  * * *
28  * * *
29  * * *
30  * * *
```

上面`traceroute`工具输出了很多行结果，其中每一行都是数据包需要经过的网关节点，从上面可知，我们访问`www.bing.com`数据包总共需要30跳。

每一行的内容是`节点名（IP） 第1次ping延时 第2次ping延时 第3次ping延时`，默认情况下`traceroute`工具会对网关节点`ping`3次。显示为`* * *`的行代表该节点关闭了ICMP ECHO的响应功能，不允许被`ping`。

## 常用参数

* `-n`：默认情况下`traceroute`会自动通过反向DNS查询探测网关节点IP的主机名，如果我们不需要该功能可以通过指定`-n`选项关掉，以减少DNS查询造成的延时。