# Eclipse 安装插件的正确姿势

Eclipse内置了Market Place，理论上我们只要在里面搜索插件，然后点击`Install`就能一键安装，但事实不是这样，如此操作你可能根本找不到想要的插件，找到了也基本会安装失败，Eclipse Market Place这个东西bug实在太多：

1. 插件明明存在，但是搜索经常搜不到
2. 国内连接Eclipse的插件下载服务器极慢
3. Eclipse的插件下载机制写的极差
4. Eclipse Market Place有个持续了好几个版本的bug，安装时会报错`The following solutions are not available`

所以安装Eclipse插件是个比较玄学的过程。

## 安装步骤

### step1 找到插件地址

插件地址可以从Market Place搜出来（搜到不要直接装），或者网上找到，就是类似这样的地址：`https://download.eclipse.org/windowbuilder/latest`。

### step2 分别尝试原地址和镜像

我们先尝试用镜像安装插件，以中科大镜像为例，将`https://download.eclipse.org/windowbuilder/latest`替换成`https://mirrors.ustc.edu.cn/eclipse/windowbuilder/latest`，然后尝试安装。镜像我们可以多找几个进行尝试。

如果所有镜像均报错，或者你的代理稳定而且速度快，就可以采用原来的地址安装，使用eclipse官方地址安装时一定要使用代理服务器进行加速，不然肯定超时。Eclipse内置的代理配置似乎对于安装插件不好使，我们直接使用系统全局的代理。

### step3 运行安装程序

点击`Help->Install New Software`，分别尝试输入各种镜像和原地址运行安装程序，直到能正确安装插件即可。
