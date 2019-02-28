# Windows10 修改hosts文件

hosts文件路径

```
C:\Windows\System32\drivers\etc\hosts
```

修改时注意备份，注意修改后刷新DNS缓存：

```
ipconfig /flushdns
```

不刷新修改不起效，因为Windows10会优先读取DNS缓存。
