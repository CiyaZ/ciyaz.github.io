# ngxlua执行流程

Nginx在处理HTTP请求时，处理过程被划分为了很多阶段。`ngx_lua`中，根据这些请求阶段，划分了能够用Lua脚本处理的接口。

![](res/1.png)

其中：主要分为8个阶段：

* `init_by_lua`：初始化配置和模块
* `set_by_lua`：初始化Nginx变量
* `rewrite_by_lua`：重写URL阶段
* `access_by_lua`：访问控制等
* `content_by_lua`：产生响应内容
* `header_filter_by_lua`：组装响应头
* `body_filter_by_lua`：组装响应体
* `log_by_lua`：会话完成后记录日志等后处理

我们的脚本一般都是挂载到这些阶段的。前面我们使用`content_by_lua_block`，其实就是将一段逻辑挂载到了`content_by_lua`阶段。

## 挂载处理脚本

前一篇笔记中，我们使用`content_by_lua_block`，在`nginx.conf`中内联插入了一个Lua脚本，实际开发中，我们一般不会这样将Nginx配置文件和Lua脚本混写。

这里，我们将配置文件和脚本分开来编写。

```lua
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        location / {
            default_type text/html;

            content_by_lua_file 'scripts/hello.lua';
        }
    }
}
```

```lua
ngx.print('Hello, world!')
```

`content_by_lua_file`，表示`content_by_lua`阶段，使用文件`scripts/hello.lua`来进行处理。
