# Git无法识别文件名大小写问题

今天被这个问题狠狠的坑了一把。

## Windows下文件名大小写问题

Windows下，我们可以将`a.txt`修改为`A.txt`，将这个文件移动到Linux文件系统下，大写小写都能正确识别。

但是我们不能让一个文件夹中，同时存在`a.txt`和`A.txt`，它们被Windows认为是相同的文件名，会相互覆盖，这是历史原因造成的。而Git for Windows遵循了这个奇葩的历史设计。

## Git在Windows下文件名之坑

### 坑1

由于Windows下文件名大小写的奇葩设计，Git for Windows默认不会提交文件名大小写更改，因而Linux下的版本库也无法同步这些更改。

设定Windows下启动Git大小写敏感：
```
git config --global core.ignorecase false
```

全局设定后，当前版本库的设置不会变，所以还需要额外设置一次当前版本库的配置：
```
git config core.ignorecase false
```

## 坑2

如果你的版本库中，同时存在了`a.txt`和`A.txt`，很好，Windows下Git根本就无法正常工作。
