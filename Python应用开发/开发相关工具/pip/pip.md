# pip 包管理器

pip是Python的包管理工具，Python最大的亮点之一，就是丰富的第三方库支持。pip几乎是学习Python必备的工具，在Windows上安装最新的Python3.6，pip会一同自动安装。Linux下，系统默认预装Python和pip工具。

下面我们介绍一些pip的常用命令，以备将来快速查阅。本文没有记录的命令选项，记得查阅`man`手册。

注意：Linux下，由于系统预装了Python2和Python3，因此同时预装了两个版本的pip，Python3对应的pip命令为pip3，这和Windows下有点区别。

## pip和PyPI

PyPI(the Python Package Index)，是Python第三方扩展的软件仓库。类似Maven中央仓库的性质，pip就是从PyPI下载软件包的。PyPI在国内访问是比较快的，一般不需要镜像源。

### easy_install

除了pip之外，Python还有一个包管理工具easy install，不过现在不是很常用了，可以说pip解决了我几乎全部的问题。我上次使用easy install是几年前了，那时我还使用Python2，用pip安装了一个软件包，结果遇到了bug，神奇的是尝试用easy install结果就正常了。

### whl

现在Python包管理很流行whl格式包（wheel，轮子），whl是一种新的包管理打包标准，旨在取代eggs（另一种打包格式），由于Windows下用户可能没有安装编译器（MSVC），很多Python模块编译不了，whl是预编译的，因此Windows下使用whl很方便。Linux下无所谓，因为几乎所有桌面发行版都内置了GNU工具链，或者安装这些工具也是十分方便的。

安装whl
```
pip install xxx.whl
```

## 安装软件包

```
pip install <package>
```

注：该命令在Linux下，会先下载文件，然后可能会编译C语言模块（如果有C代码）。全局安装千万不要忘了`sudo`，不然苦苦编译20分钟，结果`permission denied`安装失败，又要重来了。

* `--upgrade`参数：升级软件包

## 卸载软件包

```
pip uninstall <package>
```

## 从PyPI查找软件包

```
pip search <package>
```

该命令会查找和指定名字相似的软件包。

## 查看已安装的软件包的信息

```
pip show <package>
```

该命令会详细列出已安装的软件包的信息。

## 查看已安装的所有软件包

```
pip list
```

* `--outdate`参数：列出已过时，需要升级的软件包
