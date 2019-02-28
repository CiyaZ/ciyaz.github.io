# APT

apt是Debian及其衍生发行版使用的包管理机制，我们最熟悉的Ubuntu，LinuxMint就是使用的apt。debian会维护一个远程的软件资源服务器，称为软件源，我们的桌面电脑需要同步其索引存储在本地，称为包管理数据库，并在需要的时候从服务器下载软件的安装包。

apt系列命令实际管理的是包管理数据库。

### apt-get

* apt-get update 刷新本地软件包数据库信息
* apt-get upgrade 更新所有可更新的软件包
* apt-get dist-upgrade 大幅更新系统软件包，通常用于系统升级，例如系统大版本更新或将stable源切换到test源
* apt-get install <pkg> 安装软件包，如果是已安装的软件包，检查源中是否有新版本，如果有，就更新它
* apt-get -f install 修复依赖，由于某些非正常原因（比如强制手动删除了一个软件包）可能导致依赖树破损，通常不会出现，如果出现系统会提示我们执行这条命令
* apt-get remove <pkg> 删除软件包
* apt-get purge <pkg> 彻底删除软件包，和remove不同的是，remove会保留一部分配置文件，purge会按照deb包的清单文件删除所有安装文件
* apt-get clean 我们安装软件包后，软件的安装deb包会在本地缓存，clean可以清除这些安装包缓存

### apt-cache

* apt-cache search <regex> 查找软件包
* apt-cache show <pkg> 显示软件包详细信息
* apt-cache depends 显示一个软件包的依赖信息
* apt-cache redepends 显示哪些软件包依赖此软件包

## dpkg命令

dpkg命令也可以用于包管理，但是要注意，使用dpkg安装的软件包会绕过apt包管理数据库。它负责的只是实际的软件包安装与卸载。

* dpkg -i <pkg> 安装软件包
* dpkg -l 列出已安装软件包
* dpkg -r 删除软件包
* dpkg -P 彻底删除软件包
* dpkg -L <pkg> 列出软件包都安装到哪里去了

apt常用配置

* /etc/apt/sources.list 软件源信息
