# message-消息

Web开发中有一个概念叫`flash message`，它用来设置一条信息，这条信息存储在Session或Cookie中，在当前会话下次HTTP请求的响应中返回这条信息给页面。这个功能一般用来输出表单错误信息等。Django中默认集成的`message`组件能够实现这个功能。

## 例子

下面代码中，我们编写这样一个测试功能：首先访问`/add_msg`添加一条消息，然后访问`/show_msg`回显这条消息，再次访问`/show_msg`因为消息已经被消费掉，应该没有任何显示。

```python
from django.http import HttpResponse
from django.shortcuts import render
from django.contrib import messages


def add_msg(request):
    messages.add_message(request, messages.INFO, '测试消息')
    return HttpResponse('OK')


def show_msg(request):
    return render(request, 'msg.html')
```

上面代码中，我们调用`add_message`设置了消息，其中，`message.INFO`是消息级别，Django中消息和日志错误级别类似，也有一个消息级别，这里使用的是`INFO`级别。

```html
{% if messages %}
    {% for message in messages %}
        {{ message.tags }} | {{ message }}
    {% endfor %}
{% endif %}
```

在页面模板中，我们可以直接通过`messages`变量访问消息，我们遍历消息变量，其中`message.tags`可以获得消息级别，`message`可以获得消息内容。

![](res/1.png)
