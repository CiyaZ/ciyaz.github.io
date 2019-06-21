# Eclipse 安装插件的正确姿势

Eclipse内置了Market Place，理论上我们只要在里面搜索插件，然后点击`Install`就能一键安装，但事实不是这样，如此操作你可能根本找不到想要的插件，找到了也基本会安装失败，Eclipse Market Place这个东西bug实在太多：

1. 插件明明存在，但是Market Place搜索经常搜不到
2. 国内连接大部分插件下载服务器都极慢，有的服务器会在一段时间没能下载完时，故意关闭连接，导致整个安装过程失败
3. Eclipse的插件下载机制写的极差，单个文件是单线程下载，不能断点续传，更没有P2P
4. Eclipse Market Place有个持续了好几个版本的bug，安装时会报错`The following solutions are not available`，然后安装失败

所以安装Eclipse插件是个比较玄学的过程。

## 安装步骤

### 搜索插件

首先尝试在Market Place中搜索插件，如果没有，在网页版中搜索，因为有些在Market Place中搜不到：

[https://marketplace.eclipse.org/](https://marketplace.eclipse.org/)

Eclipse可以识别网页版的下载链接，因此安装比较简单，直接将`Install`按钮拖拽到Eclipse中就行。

有些插件没有在Market Place中发布，如果有插件地址，直接点击`Help->Install New Software`，安装就行了。

### 安装失败

Eclipse安装一个插件时会下载大量小文件，这些小文件有一个下载失败就会导致整个安装过程失败，好在之前下载的数据不会清空，如果下载报错，我们直接把报错的文件手动下载下来，放到对应的路径，一般是Eclipse的`plugins`或`features`目录。

### 尝试迅雷或镜像

针对上面安装失败的状况，可能是由于某个文件下载非常困难，这非常不建议用Chrome浏览器再尝试下载，Chrome的下载机制写的也极差，而且从未改过，不适用于国内。迅雷和内置迅雷的360浏览器能够断点续传和多线程下载，如果实在不行，可以尝试从镜像中寻找。

以中科大镜像为例，将`https://download.eclipse.org/windowbuilder/latest`替换成`https://mirrors.ustc.edu.cn/eclipse/windowbuilder/latest`，寻找对应文件。

### 尝试代理

如果实在无法下载，镜像中也没有，那就只能尝试代理了。然而随着网络封锁越来越严，经济又好用的代理是很难实现的，这里就不多说了。
