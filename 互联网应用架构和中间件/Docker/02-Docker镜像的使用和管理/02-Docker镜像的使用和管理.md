# Docker镜像和使用和管理

## 下载Docker镜像

我们可以在DockerHub搜索别人已经封装好的镜像。

获取镜像的命令：
```
docker pull [选项] [仓库地址]<仓库名>:<标签>
```

* 仓库地址：默认是DockerHub，手动指定私有仓库一般是`IP:端口号`的形式
* 仓库名：仓库名是两段式的，即`用户名/软件名`，对于DockerHub，如果不给出用户名，默认是`library`，指官方镜像

这里我们下载一个Ubuntu14.04的镜像。

```
ciyaz@ubuntu:~$ docker pull ubuntu:14.04
14.04: Pulling from library/ubuntu
aa1a66b8583a: Pull complete
aaccc2e362b2: Pull complete
a53116a2808f: Pull complete
b3a7298e318c: Pull complete
Digest: sha256:f961d3d101e66017fc6f0a63ecc0ff15d3e7b53b6a0ac500cd1619ded4771bd6
Status: Downloaded newer image for ubuntu:14.04
```

## 使用Docker镜像

这里我们运行一下刚刚下载的Ubuntu镜像：

```
docker run -it --rm ubuntu:14.04 bash
```

* `run`：run命令表示创建并执行容器
* `-i`和`-t`：启动交互式终端
* `-rm`：容器退出后将其删除，一般情况下出于排障需求，容器退出后不立即删除
* `bash`：放在镜像名后的是命令，这里我们指定启动个bash

`docker run`还有许多其它参数，针对不同的适用场景，具体请参考文档。

## 列出镜像

```
docker images
```

这里我们可以看到刚才下载的Ubuntu镜像：

```
ciyaz@ubuntu:~$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               f17b6a61de28        8 days ago          188MB
```

## 删除镜像

```
docker rmi <镜像>
```

有实例化容器的镜像不能被删除，必须先删除容器（命令为`docker rm`），才能删除镜像。
