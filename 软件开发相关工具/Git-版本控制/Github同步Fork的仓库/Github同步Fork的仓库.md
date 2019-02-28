# Github同步Fork的仓库

我们有时候在Github上Fork了别人的项目，但是别人的项目还在更新，我们怎么保证我们的远程仓库和别人仓库里的代码保持同步呢？

下面是一个简单的例子

```shell
git remote add upstream git@github.com:xxx.git
git pull upstream master
git push origin master
```

直接添加一个叫upstream的远程仓库，然后pull，合并冲突后push到自己的仓库就行了。
