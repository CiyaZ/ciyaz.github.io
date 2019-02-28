# Docker 容器操作

这篇笔记我们学习如何创建、启动、停止、删除Docker容器。

Docker围绕镜像和容器，有一系列的管理命令，网上有这样一张图非常好：

![](res/1.png)

## 新建并启动容器

```
docker run <镜像名> [命令和参数]
```

* `-t`和`-i`：分别是启动终端并对接到容器的标准输入上，和保持标准输入打开，这两个选项通常一起出现
* `-d`：后台运行，网络服务一般都是指定这种形式运行
* `-p`：端口映射，格式例如`-p 80:8080`，表示将容器的8080端口映射到宿主机的80端口
* 如果命令不存在，就使用Dockerfile中CMD指定的默认命令

运行`docker run`后，Docker为我们执行了的操作：

1. 检查指定镜像本地是否存在，不存在则从Docker仓库下载
2. 基于镜像创建并启动一个容器
3. 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
4. 从宿主机的网桥接口中桥接一个到容器中，并给容器分配一个IP
5. 执行用户指定的程序，如果不存在执行默认程序，程序执行完毕，容器退出

## 查看所有容器状态

```
docker ps
```

* `-a`：表示查看所有已运行和未运行的容器

例子：
```
ciyaz@ubuntu:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                  NAMES
2e07b8fccd05        sb-demo             "java -jar /app.jar"   6 seconds ago       Up 4 seconds        0.0.0.0:80->8080/tcp   elastic_nightingale
```

* `CONTAINER ID`是容器的ID，出于方便考虑，我们可以取其特有的前几位作为操作容器的ID参数，比如上面我们直接使用`docker start 2e0`就能操作这个容器，而不用把`2e07b8fccd05`全部输进去
* `STATUS`是容器的运行状态，如果未运行会显示`Exited`

## 查看一个容器的详细配置信息

```
docker inspect <容器ID>
```

输出结果是一个json。

## 进入容器shell

```
docker exec -it <容器ID> /bin/sh
```

该命令会在Docker容器中再启动一个shell前台进程，供我们对容器进行操作。

## 启动已经终止的容器

```
docker start <容器ID>
```

容器ID可以使用`docker ps -a`查看，取其特有的前几位即可。注意，这个容器的运行参数在第一次`docker run`时已经制定了，包括后台运行`-d`，端口映射`-p 80:8080`，这些参数在`docker start`时就不能再重新指定了。

## 停止容器

```
docker stop <容器ID>
```

## 重启容器

```
docker restart <容器ID>
```

## 查看后台运行容器的输出

前面我们知道可以使用`docker run -d`指定容器后台运行，此时如果我么想要查看容器的标准输出内容，可以使用`docker logs`。

例子：
```
ciyaz@ubuntu:~$ docker logs 2e0

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.0.RELEASE)

2018-11-29 01:02:55.893  INFO 1 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication v0.0.1-SNAPSHOT on 2e07b8fccd05 with PID 1 (/app.jar started by root in /)
2018-11-29 01:02:55.905  INFO 1 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
...
```

## 删除容器

```
docker rm <容器ID>
```

运行中的容器是无法删除的。

注意：删除镜像是`docker rmi`，删除容器是`docker rm`，不要搞混了。

## 查看容器运行的资源占用

```
docker stats
```

该功能类似`top`，会显示所有Docker容器的内存占用，CPU占用等情况。
