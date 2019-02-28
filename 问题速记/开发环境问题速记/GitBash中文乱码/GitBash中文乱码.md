# Git Bash 中文乱码

问题发生在Windows版的`Git Bash`上，如图所示

![](res/1.png)

注：Linux下新版的Git好像也同样有这个问题。

# 解决方法

```
git config --global core.quotepath false
```
