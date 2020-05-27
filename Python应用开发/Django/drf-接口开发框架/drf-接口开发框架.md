# drf-接口开发框架

Django REST Framework（以下简称drf）是Django框架中，用来开发Web接口的一个插件框架。

[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)

## drf简介

通过前面学习我们知道，Django的一个缺点就是前后端高度耦合，甚至出现了`form`组件这种邪路，做些个人博客、个人小型CMS之类的站点确实方便，但这已经无法满足现代较大型应用系统的建设需求了。

现在前端的发展非常迅速，早已经不是撸几个静态页，每个页面写千十来行难以维护的JQuery代码的时代了，前端项目的工程化和用户体验也得到了极大提升，现在后端更应把精力放在接口和业务逻辑上。

Django本体在这方面并没有多考虑，因此直接使用Django编写Web接口，需要写很多非业务功能性的重复代码，浪费时间，drf就是为了解决这个问题而设计的。

## drf的主要功能

drf主要功能分两部分：

1. 序列化器：使用drf的序列化器能够很方便的将查询模型的结果集映射到JSON。
2. 视图模板：drf提供了GenericAPIView和一系列Mixins，很多增删改查其实我们不需要手写，直接声明式的配置相关组件就可以自动生成相关功能了。

注：drf中mixins是指一种能够由视图继承的类，它们以REST规范实现了各种常用功能，比如查询列表、更新模型等，我们的视图采用多继承的方式继承这些mixins组件，就能自动实现相应功能。

当然，实际开发中肯定会碰到内置mixins无法满足的时候，我们基于drf的序列化器，手动写自己的视图也是很常见的状况。

## 配置drf框架

Django默认没有内置drf框架，我们需要手动安装：

```
pip3 install djangorestframework
```

然后在`settings.py`中，加入相关配置：

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    ...
]
```

## 使用drf响应JSON数据

这里我们编写一个简单的例子工程`books`，使用drf的序列化器和响应组件，返回一组JSON数据。

下面例子中，我们有三个Model，书籍（Book）、分类（Category）、作者（Author）。

![](res/1.png)

模型代码就不再多说了，这里我们关注`serializers.py`，它是我们定义drf序列化器的文件。

serializers.py
```python
from rest_framework import serializers
from books.models import *


class CategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = Category
        fields = ['pk', 'name']


class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Author
        fields = ['pk', 'name', 'gender']


class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['pk', 'name', 'isbn', 'publish_time', 'category', 'authors']
```

里面内容其实很简单，就是模型配置到序列化器上。这里我们使用的是`ModelSerializer`，它是一个相对比较通用的序列化器，能够直接将指定的模型字段映射为JSON。

views.py
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from books.models import *
from books.serializers import *


@api_view(['GET'])
def get_book_list(request):
    book_list = Book.objects.all()
    rsp_data = BookSerializer(book_list, many=True)
    return Response({
        'rspCode': 0,
        'rspMsg': '查询成功',
        'data': rsp_data.data
    }, status=200)

```

在`views.py`中，我们编写了一个`Function based view`，注意`@api_view`这个装饰器，她是drf提供的，用于处理drf专用请求响应组件，使用drf时我们需要带上这个装饰器（注：后面会用到Class based view，那边会使用另一种写法）。

代码逻辑很简单，我们查询出结果列表后，将其传入序列化器。`many=True`这个关键字参数意味着返回的结果是多个而不是一个。

最后返回一个drf提供的`Response`对象，其中第一个参数是一个字典类型，代表返回的JSON数据，其中`rsp_data.data`是序列化结果的字典类型数据。`Response`中比较常用的关键字参数`status`，是我们可以自定义的HTTP响应码。

未完，待续
