# JQuery ajax

javascript的XMLHttpRequest可能不够方便，因此JQuery提供了封装，简化了ajax操作。

## load()

```javascript
$(selector).load(URL,data,callback)
```

load()从服务器加载数据并把返回结果放入被选元素中，URL是请求资源地址，可选参数data是查询参数字符串键值对集合，可选的callback是执行完成的回调。

回调函数参数：responseTxt是调用成功的结果内容，statusTxt是调用的状态（取值"success","error","timeout","notmodified"），xhr是XMLHttpRequest对象。

## ajax()

JQuery发起Ajax请求建议使用`$.ajax()`

使用实例：

```javascript
$.ajax({
  type: "POST",
  async: true,
  url: '/backend_writing',
  data:  requestStr,
  dataType: "json",
  contentType: "application/json; charset=utf-8",
  success: function (msg)
  {
    if(msg.result === 'success')
    {
      modal_div.text('操作成功');
      open_modal();
    }
    else if(msg.result === 'failed')
    {
      modal_div.text('操作失败 服务器内部错误');
      open_modal();
    }
    else
    {
      modal_div.text('操作失败 未知错误');
      open_modal();
    }
  },
  error: function (err)
  {
    modal_div.text('操作失败 网络错误');
    open_modal();
  }
});
```

上面例子来自一个cms项目，当时在写这个ajax的时候，试图使用`$.post()`向SpringMVC后台传递json，但是json却被URL编码当成表单处理，设置数据类型也不行，而且回调还总是不执行。所以建议JQuery的Ajax全部使用`$.ajax()`。

这里要注意，后台返回的数据类型也必须是能被正确解析的json类型，msg就是返回json对应的javascript对象，否则回调的是error函数，返回信息在err变量中，这不符合这个函数设计的初衷。

补充：后台返回的JSON中，键值和字符串需要用双引号，否则格式错误，`success`是不会回调的

## jsonp类型

如果需要进行跨域ajax请求，JQuery为我们封装了jsonp类型数据的ajax请求，使用非常简单，只需要将`dataType`改为`jsonp`，请求会随机生成一个`callback`参数。

* 如果需要自定义`callback`参数的值，加上`jsonpCallback`属性进行指定
* 如果请求参数名不叫`callback`，再加一个`jsonp`属性进行指定

## get()/post() 不推荐使用

发起GET请求：

```javascript
$.get(url, callback,dataType)
```

dataType是数据类型，可以是"txt"，"html"，"json"，"xml"，jquery会自动解析，当选择"json"时，会直接返回javascript对象

发起POST请求：

```javascript
$.post(url, data, callback,dataType)
```

data是请求数据

回调函数参数：data 响应数据，status 响应状态（取值"success","error","timeout","notmodified"）
