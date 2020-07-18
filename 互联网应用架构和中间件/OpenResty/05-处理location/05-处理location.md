# 处理location

Nginx中，可以通过定义`location`来设定静态文件、反向代理等功能的路径，OpenResty中则可以使用Lua脚本，来调用定义好的`location`。

## ngx.exec() 内部重定向

`ngx.exec()`能够在处理流程内部，将请求重定向到另一个`location`上。

```lua
return ngx.exec('/api')
```

注意：`ngx.exec()`不会直接中断处理流程，因此这里加上了`return`。

## ngx.redirect() 重定向

`ngx.redirect()`会向客户端发送`302`重定向响应。

```lua
return ngx.redirect('/api')
```

和`ngx.exec()`区别也很明显，`ngx.redirect()`的重定向是告知客户端后，由客户端发起的。

## ngx.location.capture() 调用子请求

这里我们编写了一个简单的接口`/add?a=x&b=y`，能够读取HTTP请求参数`a`和`b`，然后计算相加的结果。

```lua
local args = ngx.req.get_uri_args()
local a = args.a
local b = args.b
if a == nil then
	a = 0
end
if b == nil then
	b = 0
end
-- 这里为了观察并行和串行调用区别，睡眠了1秒
ngx.sleep(1)
ngx.print(tostring(a + b))
```

串行调用子请求：

```lua
local res1 = ngx.location.capture("/add", {args={a=1, b=2}})
local res2 = ngx.location.capture("/add", {args={a=3, b=4}})
ngx.say("status:", res1.status, " response:", res1.body)
ngx.say("status:", res2.status, " response:", res2.body)
```

此时，调用需要2秒返回。

串行调用用于有前后依赖的多个子请求。

并行调用子请求：

```lua
local res1, res2 = ngx.location.capture_multi( {
				{"/add", {args={a=1, b=2}}},
				{"/add", {args={a=3, b=4}}}
			})
ngx.say("status:", res1.status, " response:", res1.body)
ngx.say("status:", res2.status, " response:", res2.body)
```

此时，调用需要1秒返回。

我们发现，对于没有前后依赖的子请求，我们可以使用并行调用，大大缩短响应速度。
