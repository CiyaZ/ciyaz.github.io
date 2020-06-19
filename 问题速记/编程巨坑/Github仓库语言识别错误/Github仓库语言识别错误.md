# Github仓库语言识别错误

Github的一个缺点是不能设置语言，而是通过一个叫`linguist`的功能自动识别仓库中代码使用的语言，但是这个`linguist`很不智能，给我们造成了很多麻烦。比如：写了一个后端Java功能，由于添加了一些CSS库，结果仓库就被识别为了CSS语言，这种做法十分荒谬。但是我们可以用`.gitattributes`文件覆盖掉一些`linguist`的配置。

例子，指示Github把几个前端库忽略掉：
```
src/main/resources/static/easyui-1.6.2/* linguist-vendored
src/main/resources/static/echarts/* linguist-vendored
src/main/resources/static/jquery-3.3.1/* linguist-vendored
```

这样Github就只会识别我们自己编写的代码，我们仓库的语言显示就正常了。

## 网上的一些说法

网上有一些博客，互相抄来抄去抄了几十篇，简直是到了污染网络环境的地步。他们的做法类似这样的：
```
*.js linguist-language=Java
```

这意味着把所有`JavaScript`源码当`Java`算，显然是不对的，我们仓库的源代码语言统计比例也会明显错误，这里建议还是忽略掉所有库目录。
