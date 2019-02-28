# 处理HTTP请求

django中，解析HTTP请求主要是由框架自动完成的，我们只需要在`urls.py`定义处理的规则。django很容易实现传统的URL风格，或是RESTful的URL风格。django主要面向小型的互联网应用，因此RESTful风格的URL很受欢迎，但我个人不太喜欢，这里传统方式和REST方式都介绍一下，用起来都比较简单。

## 获取GET参数

请求示例
```
http://localhost:8080/app1?id=1
```

urls.py
```python
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^$', views.index),
]
```

views.py
```python
from django.shortcuts import render
from django.http import HttpResponse


def index(request):
	id = request.GET.get("id")
	return HttpResponse(id)
```

GET参数是通过`request.GET`这个QueryDict对象传入的，可以通过`get()`方法或是索引方法获取，如果键不存在，get返回None。POST同理，这里就不多介绍了。

## 获取RESTful风格的请求参数

请求示例
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

如果用户访问时，没有传递id参数，就会抱404。
