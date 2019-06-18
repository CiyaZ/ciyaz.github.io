# Django部署

我们开发时使用的`runserver`命令性能是远远无法满足生产环境需求的，这篇笔记介绍如何在Linux服务器上部署Django项目。这里Django使用的是`uwsgi`处理动态请求，由Nginx进行转发以及处理静态资源请求。

注：Linux发行版使用的是UbuntuServer16.04。

## 依赖安装

假设这里已经安装好了Nginx、MySQL等常用组件。由于Python的`mysqlclient`还依赖`libmysqlclient-dev`这个包，在安装MySQL时它默认是没有安装的，因此也要将其额外安装。

安装pip（包管理器）：
```
sudo apt-get install python3-pip
```

安装Python相关依赖：
```
pip3 install django mysqlclient
```

安装uwsgi服务：
```
pip3 install uwsgi
```

## 配置uwsgi

我们参考Django官网和uwsgi官网进行配置。

这里假设我们的工程名为`blog`，在工程根目录存在`blog/wsgi.py`文件。在`uwsgi`配置文件中，需要指定该文件的路径。

```ini
[uwsgi]
chdir = /home/ubuntu/blog/
wsgi-file = /home/ubuntu/blog/blog/wsgi.py
processes = 4
threads = 2
http = 127.0.0.1:8080
```

启动`uwsgi`服务：

```
uwsgi --ini uwsgi.ini
```

## 配置nginx

这一步就比较简单了，方法也不唯一。我们这里主要是对动态请求的转发和静态请求的路径处理。注：静态资源位于工程根目录的`static`文件夹下。

```
location / {
  proxy_pass http://127.0.0.1:8080;
}

location /static/ {
  root /home/ubuntu/blog/;
}
```

## 进行数据迁移

```
python3 manage.py makemigrations <app名>
python3 manage.py migrate
```

注意：工程中的数据库配置文件需要正确配置所使用的数据库，MySQL中要预先将数据库建好。

这样，访问Nginx的服务地址，我们的Django工程就部署好了。

## 使用守护进程启动uwsgi

前面我们启动uwsgi的方式并非守护进程，一般我们都是将其启动为守护进程，在后台运行。uwsgi已经提供了这个功能，不需要`nohup`之类的命令。只要在配置文件中加入一行即可：

```
daemonize = /var/log/uwsgi/blog.log
```

注：`daemonize`中配置的值是日志文件路径。
