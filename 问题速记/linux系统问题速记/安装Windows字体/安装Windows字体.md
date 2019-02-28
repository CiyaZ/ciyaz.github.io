# Ubuntu下安装Windows字体

由于版权原因，Ubuntu下自带的字体比较少，我们可以把Windows系统下的字体拷贝过来使用。

Windows下，字体在`C:\\Windows\\Fonts`目录，把整个目录打包发给Ubuntu系统。

Ubuntu下，将这个目录解压到`/usr/share/fonts/`下，最好把文件夹重命名一下，以进行区分，比如命名为`ms-fonts-cpy`。

进入新的字体目录执行以下命名：

```
sudo chmod 744 *
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -f -v
```

设置字体目录和刷新字体缓存时会稍微卡一下，要耐心等待。这样执行完成后，注销并重新登录，就能使用新的字体了。
