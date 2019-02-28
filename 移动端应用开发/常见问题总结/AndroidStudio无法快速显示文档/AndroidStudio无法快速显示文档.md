# Android Studio 无法快速显示文档

Android Studio 下，使用文档快捷键，一直提示Fetching Documentation，过了半天文档才能正常显示，此现象可能是某次更新造成的bug。

当前版本是Android Studio2.3.3，在我的Linux开发机和Windows电脑上都存在这个问题。

## 解决办法

首先确保SDK中已经安装了文档。

![](res/1.png)

Linux下修改`~/.AndroidStudio/config/options/jdk.tab.xml`，Windows下也是对应目录：把所有的

```xml
<root type="simple" url="http://developer.android.com/reference/" />
```

替换成
```xml
<root type="simple" url="file:///opt/Android/Sdk/docs/reference" />
```

注意XML标签闭合，这个文件有一点语法错误，Android Studio都不能正确识别SDK。
