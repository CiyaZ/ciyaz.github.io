# 生成HTTP响应

HTTP响应主要内容包括状态码、响应头和响应体，这里我们来学习OpenResty中，如何生成HTTP响应。

## 输出响应头

添加响应头，只需要向`ngx.header`添加内容即可。下面是一段例子代码：

```lua
local cjson = require 'cjson'

local rsp_json = {name = 'Tom', age = 18}
ngx.header.content_type = 'application/json'
ngx.print(cjson.encode(rsp_json))
```

代码中，我们设置了响应头`Content-Type`为`application/json`。

## 输出响应内容

之前我们已经使用过`ngx.say()`和`ngx.print()`，两者的区别是前者会在输出结果末尾加一个换行符，便于调试。

我们输出XML或是Json，一般都是使用`ngx.print()`。

## 输出状态码

`ngx.exit()`可以立即中断一个处理流程，并返回一个响应码。其中，常用的响应码被定义为了常量，我们可以直接使用。

```lua
ngx.exit(ngx.HTTP_BAD_REQUEST)
```

上面代码中，返回了HTTP 400。
