# Django部署

我们开发时使用的`runserver`命令性能是远远无法满足生产环境需求的，这篇笔记介绍如何在Linux服务器上部署Django项目。这里Django使用的是`uwsgi`处理动态请求，由Nginx进行转发以及处理静态资源请求。

Django的部署步骤是比较多的，这里我们按照最佳实践的顺序来依次介绍，一般按照步骤操作就不会有什么问题。

注：Linux发行版使用的是UbuntuServer16.04。

## 依赖安装

### Nginx相关

通常来说，我们是需要一个Nginx来转发请求，它负责将静态资源URL映射到静态文件上，将动态请求映射到uwsgi服务上。

Nginx安装和Django没什么关系，我们可以直接参考Nginx或OpenResty相关章节。

### MySQL相关

大多数应用都使用MySQL数据库。由于Python的`mysqlclient`还依赖`libmysqlclient-dev`这个包，在安装MySQL时它默认是没有安装的，因此也要将其额外安装。

安装pip（包管理器）：
```
sudo apt-get install python3-pip
```

安装Python相关依赖：
```
pip3 install django mysqlclient
```

如果你的Django工程使用其他数据库，则需要安装对应的依赖，这里就不多介绍了。

### uwsgi相关

安装uwsgi服务：
```
pip3 install uwsgi
```

### 其他依赖

如果工程还依赖其他扩展，则需要依次手动安装。

## 进行数据迁移

数据迁移前，首先确认工程的数据库配置是否正确，服务器中数据库是否已经建好，文本编码是否正确（一般都是采用`utf8mb4`）。

确认无误后，使用如下命令检查并生成新的数据迁移：

```
python3 manage.py makemigrations <app名>
python3 manage.py migrate
```

## 发布静态资源

使用Nginx转发静态资源时，我们需要用`collectstatic`将所有静态文件配置到静态资源文件夹。这里所说的静态文件，常见的包括：我们编写的CSS、JS文件，DjangoAdmin面板的CSS、JS文件，DRF接口预览页面的CSS、JS文件等等。`collectstatic`能将所有这些文件合并放到一个文件夹。

```
python3 manage.py collectstatic
```

注：使用该命令，需要在工程配置中指定`STATIC_ROOT`，该配置可以用绝对路径指定到工程外部，比如`/var/www/html/blog/static/`。

如果你的Django工程完全是个后端接口项目，则不需要该步骤。

## 配置uwsgi

我们参考Django官网和uwsgi官网进行配置。

这里假设我们的工程名为`blog`，在工程根目录存在`blog/wsgi.py`文件。

我们这里可以在任意位置编写一个`uwsgi.ini`（我的习惯是搭建项目时在工程中写好）：

uwsgi.ini
```ini
[uwsgi]
chdir = /home/ubuntu/blog/
wsgi-file = /home/ubuntu/blog/blog/wsgi.py
processes = 4
threads = 2
http = 127.0.0.1:8080
daemonize = /var/log/uwsgi/blog.log
```

启动`uwsgi`服务可以使用如下命令：

```
uwsgi --ini uwsgi.ini
```

注：配置文件中`daemonize`指定了`uwsgi`使用守护进程模式，以及输出日志位置，我们不需要手动使用`nohup`等命令。

## 配置nginx

这一步就比较简单了，方法也不唯一。我们这里主要是对动态请求的转发和静态请求的路径处理。注：这里我的静态资源发布到了`/var/www/html/blog/static`文件夹下。

```
location / {
  proxy_pass http://127.0.0.1:8080;
}

location /static/ {
  root /var/www/html/blog/;
}
```

这里配置的比较简易，真实上线项目中，Nginx配置的会非常复杂，而且可能不止一层，这通常需要运维人员来专门维护。
