# Leanote

最近一直寻找一个wiz笔记的替代品，大多数都不符合我的要求，正准备自己造轮子的时候，发现了leanote，觉得比较满意。

## !!!最新补充!!!

这个软件的服务端bug还是比较多，而且迁移困难，慎用，满足一般用户还是可以的，但是我早就不用了。

## 之前尝试过的笔记软件

* 为知笔记 开始收费，Linux客户端很久没更新

* 有道云笔记 没有Linux客户端

* SpringSeed 官网都挂掉了，找不到下载链接

* laverna 完成度不高，最要命的是本地数据存储在浏览器web sql里

* Zim 风格不喜欢，界面不友好

* RedNotebook 风格不喜欢，更像是日记本不是偏学术型的笔记软件

* 自己开发：刚刚进行了简要的设计分析，觉得即使想达到基本可用的程度，工程量也是非常大的

## Leanote的优点

* 完成度较高

* 开源，可以自己部署服务器

* 文档组织比较符合我本来的想法，编辑器也比较满意，界面也是不错的

## Leanote的缺点

* golang我不熟悉，服务器如果想二次开发比较麻烦

* 客户端竟然用的electron，我觉得应该是Java或者QT

* 客户端布局显然还是有些小bug，我觉得这就是electron的问题，我不喜欢，服务端显然bug不少，而且可能难以升级维护，不过已经差不多够用了

* 竟然还没有Android客户端

* Markdown编辑器还是有点弱（leanote应该是用的现成的轮子），和富文本编辑器比起来效果不够好，看来还是得简单随笔用Markdown，复杂文档用富文本编辑器

## 安装Leanote

### 部署服务器

1. [下载](http://leanote.org/#download)预编译服务端程序，解压

2. 安装mongodb
    `sudo apt-get install mongodb`

3. 导入服务端数据库
    `mongorestore -d leanote --dir leanote_install_data`

4. 修改配置文件 `leanote/conf/app.conf`

5. 启动服务器 `leanote/bin/run.sh`，然后写一个启动脚本方便以后打开服务器。默认服务在9000端口。

6. [下载](http://app.leanote.com/)安装客户端

7. 使用默认账户名密码 admin abc123登录私有服务器，以后不开服务器打开客户端也没什么问题
