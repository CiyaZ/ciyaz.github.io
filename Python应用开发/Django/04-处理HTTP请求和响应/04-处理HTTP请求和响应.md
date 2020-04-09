# 处理HTTP请求和响应

这篇笔记介绍Django中，如何获取请求参数、请求内容，以及如何生成HTTP响应。

## 路由配置

Django和Flask、SpringMVC等框架不同，路由需要集中在几个配置文件中。

首先我们需要在`settings.py`中配置一个`ROOT_URLCONF`，这个属性指定根路由配置。下面代码指定根路由配置文件是`blog`模块的`urls.py`。

settings.py
```python
ROOT_URLCONF = 'blog.urls'
```

下面例子代码中，我们配置了几个路由作为例子，实际上，Django读取的路由配置就是一个叫做`urlpatterns`的列表。

blog/urls.py
```
from django.urls import path, re_path
from django.urls.conf import include
from blog.views.index_view import index
from blog.views.blog_view import blog


urlpatterns = [
    path('', index),
    path('index', index),
    re_path(r'^blogs/(?P<id>[0-9]+)$', blog),
    path('backend/', include('backend.urls'))
]
```

浏览器访问`/`或`/index`会由`index_view.index()`进行处理，访问`/blogs/<blogId>`会由`blog_view.blog()`响应对应的文章，而以`/backend/`为前缀的会跳转到`backend`模块的子路由配置文件。

在`backend`模块下，我们也同样创建了一个`urls.py`，其中的`urlpatterns`列表，作为该子模块的路由定义。

## 处理请求

django中，解析HTTP请求主要是由框架自动完成的，我们只需要在`urls.py`定义处理的规则。django很容易实现传统的URL风格，或是RESTful的URL风格。django主要面向小型的互联网应用，因此RESTful风格的URL很受欢迎，但我个人不太喜欢，这里传统方式和REST方式都介绍一下，用起来都比较简单。

### 获取GET请求参数

请求示例：

```
http://localhost:8080/app1?id=1
```

`views.py`获取请求参数：

```python
from django.shortcuts import render
from django.http import HttpResponse


def index(request):
	id = request.GET.get("id")
	return HttpResponse(id)
```

GET参数是通过`request.GET`这个`QueryDict`对象传入的，可以通过`get()`方法或是索引方法获取，如果键不存在，`get()`返回`None`。

### 获取POST请求参数

POST请求和GET同理，这里就不多介绍了，但是要注意，Django中默认配置了CSRF中间件，POST请求必须带上CSRF参数。该参数可以通过在表单模板中，使用`{% csrf_token %}`获取，即使是Ajax请求也需要带上该参数，否则会被拦截。

### 获取RESTful风格的请求参数

请求示例：

```
http://localhost:8080/app1/users/2
```

views.py
```python
def index_rest(request, id):
	return HttpResponse(id)
```

urls.py
```python
urlpatterns = [
	url(r'^$', views.index),
	url(r'^users/(?P<id>[0-9]+)/$', views.index_rest)
]
```

注意URL配置的写法：类似于`路径/(?P<参数名>正则表达式)/路径/`，这个写法设计的可读性不太好，不要搞乱了

如果用户访问时，没有传递id参数，那么正则匹配就不会成功，未找到合适的路由，用户就会得到`404`错误。

### 解析请求Json

我们前面了解到，Django要求POST必须带一个CSRF参数，这个参数是通过POST表单键值对的形式，提交给Django框架的，但我们一般向服务器传Json都是直接POST一个Json字符串，Django对于这种情况只会报`403`错误。要想解决这个问题，有两种方式：

方式1：以表单的形式提交CSRF_TOKEN和请求Json字符串。

```javascript
$(function () {
    var csrf = $('[name="csrfmiddlewaretoken"]').val();
    var student = {
        username: 'Tom',
        age: 18
    };
    $.ajax({
        type: "POST",
        async: true,
        url: '/test',
        dataType: "json",
        data: {
            msg: JSON.stringify(student),
            csrfmiddlewaretoken: csrf
        },
        success: function (msg) {
            console.log(msg);
        }
    });
});
```

上面代码中，首先获取`csrfmiddlewaretoken`参数，请求的JSON直接放在表单的`msg`参数中。

```python
request.POST.get('msg')
```

后台直接通过POST请求的`msg`参数，就能获取到JSON字符串了。

实际上，这种方式比较别扭，而且如果是前后端分离的开发模式，前端页面是完全不使用模板引擎解析的（通常是使用Nginx直接提供静态文件，仅对API请求进行反向代理），这样就没法从页面上获取CSRF参数了。

方式2：使用Ajax获取CSRF_TOKEN，将其加入请求header中。

Django除了支持将CSRF_TOKEN作为请求字段，还支持将其作为请求header传递给服务器，此外，我们也可以用View组件中Python代码的形式获取CSRF_TOKEN，然后通过Ajax的方式将其传给前端。

前端请求CSRF_TOKEN：

```javascript
window.onload = function () {
    $.ajax({
        type: "GET",
        async: true,
        url: '/csrftest/token',
        dataType: "json",
        contentType: "application/json; charset=utf-8",
        success: function (msg) {
            app.csrf_token = msg.csrf_token;
        },
        error: function (err) {
            alert('获取token请求失败!');
        }
    });
};
```

后端获取CSRF_TOKEN并返回：

```python
def get_token(request):
    csrf_token = csrf.get_token(request)
    return JsonResponse({
        'csrf_token': csrf_token
    })
```

前端发送POST请求传递Json数据：

```javascript
function postJson() {
    $.ajax({
        type: "POST",
        async: true,
        url: '/csrftest/post',
        headers: {
            'X-CSRFToken': app.csrf_token
        },
        data: JSON.stringify({
        'username': 'Tom',
        'password': 'abc123'
    }),
        dataType: "json",
        contentType: "application/json; charset=utf-8",
        success: function (msg) {
        },
        error: function (err) {
            alert('POST请求失败!');
        }
    });
}
```

如果是不使用模板引擎前后端完全分离开发的情况，实际上有一个CSRF_TOKEN获取时机的问题，上面代码中，我们在`window.onload()`时就获取了这个参数，除此之外，也可能存在发起POST请求前，现获取CSRF_TOKEN的情况，这时就需要注意异步顺序了。

### 处理文件上传

Django对文件上传做了封装，使用起来非常简单。

前端表单：

```html
<form method="post" action="/upload" enctype="multipart/form-data">
    {% csrf_token %}
    <input type="file" name="file" />
    <input type="submit" value="提交" />
</form>
```

注意：文件上传表单的`enctype`应该为`multipart/form-data`。

后端代码：

```python
def upload_test(request):
    fis = request.FILES.get('file')
    filename = fis.name
    fp = open('E:/' + filename, 'wb')
    for i in fis.chunks():
        fp.write(i)
    fp.close()
    return HttpResponse('OK')
```

后端代码中，我们直接通过`request.FILES.get()`就可以得到上传的文件了，这里我将其保存到了E盘根目录下（这里使用的是Windows系统）。

实际上文件上传稍有不慎就容易造成安全漏洞，上面代码非常简陋，甚至没对上传文件的类型做基本的校验，在线上环境中，应该进行充分考虑。此外，对于较大的文件，本地存储一般也不是个好主意。关于文件上传的操作，其实更常见的是调用OSS系统进行处理，这里就不多介绍了。

## 处理响应

Django中，生成HTTP响应主要分为以下几种：

1. 基本响应，可能是一个简单的字符串，或者一个404状态
2. 返回模板页面
3. 返回Json

### 返回基本HTTP响应

其实我们之前介绍文件上传时，已经用到返回一个最基本的`HTTP 200 OK`响应的方法了，下面代码向页面输出一个`OK`字符串，状态码为`200`：

```python
from django.http import HttpResponse


def index(request):
	return HttpResponse('OK')
```

返回错误状态的方法也差不多，下面代码向页面输出一个`Http 400 Bad Request`字符串，状态码为`400`：

```python
return HttpResponseBadRequest("Http 400 Bad Request")
```

其余常用的响应状态码，比如`403`，`404`，`500`等参考文档选择对应的函数即可。

### 响应模板页面

相应模板需要用到`render()`函数，它的用法其实和SpringMVC中的`ModelAndView`比较像。View组件中，我们可以返回如下内容：

```python
from django.shortcuts import render


def index(request):
	return render(request, "user_list.html", {msg: 'hello'})
```

其中，三个参数分别是`request`对象，模板文件（可以带文件夹路径），模板包含的数据对象。

### 返回重定向

对比Java中的概念，前面的代码中，view将模型交给模板进行显示，算是类似于Servlet控制器把请求转发给JSP视图或另一个Servlet（Django里不知道叫不叫转发dispatch，反正就是那个意思）。除此之外，重定向也是比较常用的，比如提交表单成功后，没什么信息可显示，需要重定向到另一个页面。

```python
from django.http import HttpResponseRedirect


def index(request):
	return HttpResponseRedirect("/app/index")
```

此时就不用`render()`了，应该用`HttpResponseRedirect()`，浏览器会根据响应自动进行跳转到对应路径。

### 返回Json

Django中虽然上传Json很复杂，但是响应一个Json很简单。下面是一个例子：

```python
from django.http import JsonResponse


def mytest(request):
    return JsonResponse({
        'username': 'tom',
        'age': 12
    })
```

其中，`JsonResponse`的参数就是返回的Json内容，这个响应Django会自动帮我们加上`Content-Type: application/json`的响应头，告知浏览器该响应是Json。
