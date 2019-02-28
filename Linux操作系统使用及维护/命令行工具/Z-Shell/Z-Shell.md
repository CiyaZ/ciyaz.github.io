# 简介

zsh比自带的bash shell有更多有趣的特性，ohmyzsh是一个很火的开源项目，安装后zsh会变的更加易用。

# 安装zsh和oh-my-zsh

```shell
#zsh
sudo apt-get install zsh
#oh-my-zsh
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

安装powerline字体，并修改终端字体为powerline。

```shell
#显示当前shell
echo $SHELL
#切换shell为zsh
chsh -s $(which zsh)
```

# 配置文件

```shell
vim .zshrc
```

修改插件、主题等。
