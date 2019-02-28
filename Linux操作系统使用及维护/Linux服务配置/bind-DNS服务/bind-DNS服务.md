# bind DNS服务

Linux下，配置DNS需要用到`bind9`这个软件，然而它设计的相当复杂且难用。通常情况下，我们需要配置DNS时，都是为了配置内网域名，下面我直接举个例子，进行一个最简单的`bind9`配置。

注意：本篇笔记并不是一个bind9教程，只是解决一个实际问题。

## 需求

假设我们需要设定一个内网域名`hcos.local`，它及它的子域名指向：

* hcos.local 192.168.1.101 一个门户页面
* gitea.hcos.local 192.168.1.101 一个Git代码库
* blog.hcos.local 192.168.1.101 一个CMS系统
* pan.hcos.local 192.168.1.101 一个网盘系统

## 安装bind9

```
apt-get install bind9
```

## 设定bind9配置文件

编辑`/etc/bind/named.conf.local`：
```
zone "hcos.local" {
	type master;
	file "/etc/bind/db.hcos.local";
};
```

* zone：比如我申请了域名`ciyaz.com`，那么我就拥有了一个域（zone）叫做`ciyaz.com.`，我可以在这个域下配置类似`blog.ciyaz.com.`，`bbs.ciyaz.com.`这样的子域

编辑`/etc/bind/db.hcos.local`：
```
$TTL	68400
@	IN	SOA	hcos.local. root.hcos.local. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	localhost.
@	IN	A	192.168.1.101
gitea	IN	A	192.168.1.101
blog	IN	A	192.168.1.101
pan	IN	A	192.168.1.101
```

注：正常情况下，我们不仅要配置DNS解析，还需要配置反向DNS解析，这里就省略了反向DNS解析的配置，只配置了正向解析，DNS服务器也是完全能够正常工作的。
