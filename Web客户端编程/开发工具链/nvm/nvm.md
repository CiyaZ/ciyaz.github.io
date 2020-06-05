# nvm

nvm用于管理安装多个版本的node环境。

对于一些历史遗留的旧工程，可能我们电脑安装的node版本太新，导致一些莫名其妙的错误。这种情况，nvm可以帮我们管理多版本node，而无需卸载新版本再安装旧版本。

Github地址：[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

## nvm安装和使用

安装nvm：
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

查看已安装的node版本：
```
nvm ls
```

查看可安装的node版本：
```
nvm ls-remote
```

安装一个版本的node：
```
nvm install [node版本号]
```

使用一个版本的node：
```
nvm use [node版本号]
```

## gnvm

对于Windows环境，我们可以使用gnvm，功能、命令和nvm类似。

[https://github.com/kenshin/gnvm](https://github.com/kenshin/gnvm)
