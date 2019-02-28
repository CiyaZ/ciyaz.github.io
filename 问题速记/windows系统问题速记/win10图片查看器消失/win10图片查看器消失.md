# win10 图片查看器消失

最新的win10系统中，微软修改了经典的“Windows照片查看器”的文件后缀名的注册表配置，这样用户打开图片时就找不到“Windows照片查看器”了，但我们修改下注册表就可以改回来。

```
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Photo Viewer\Capabilities\FileAssociations
```

![](res/1.png)

注：Windows图片查看器并不能查看有动画的`.gif`，因此这里没有把gif格式加上去。
