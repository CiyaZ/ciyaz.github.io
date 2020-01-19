# 设置Git代理

有时由于局部网络防火墙配置原因，Github等国外网站IP会被封禁，这是就需要配置代理来访问了，我们可以手动编辑`~/.gitconfig`文件。

```
[http]
	proxy = socks5://127.0.0.1:1080
[https]
	proxy = socks5://127.0.0.1:1080
```

上面配置，为Git配置了Socks5代理，将Git的`remote`访问协议改为`HTTPS`，即可正常使用了，注意`SSH`协议无法使用。
