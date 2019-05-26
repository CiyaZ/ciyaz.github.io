# Nginx部署

PHP部署相比Java是比较麻烦的，因为Java有自己的应用服务器，而PHP需要依赖Nginx，通过`php-fpm`进程在Nginx和PHP之间通信，我们知道现代Web框架都是由路由功能的，向Nginx请求的路径一般都不是某个`.php`文件的实际路径，这就涉及到Nginx的各种路径重写的问题。

## 配置文件

下面是一个在Windows平台下Nginx配置的例子，开发环境可以按照下述进行配置。

```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    server {
			listen       80;
			server_name  localhost;
			index index.php;
			root C:/Users/CiyaZ/workspace-me/moe/comic-fun/public;

			location ~ (/static/)|(favicon\.ico$)
			{
				try_files $uri =404;
			}

			location / {
				rewrite ^/(.*)$ /index.php?s=/$1;
				location = /index.php {
					fastcgi_split_path_info ^(.+\.php)(.*)$;
					fastcgi_param PATH_INFO $fastcgi_path_info;
					fastcgi_index index.php;
					include fastcgi.conf;
					fastcgi_pass 127.0.0.1:9002;
				}
		  }
    }
}
```

上面的逻辑还是很简单的，第一个`location`配置了静态资源和`favicon.ico`，这些文件不是动态资源，直接返回静态文件即可。第二个`location`会把请求转发给ThinkPHP框架的路由，这涉及到框架路由的原理，其实路由是通过`index.php`的参数`s`实现的。

生产环境中Linux服务器上也是类似的，这里就不多介绍了。
