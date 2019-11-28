# mail-发送邮件

Django内置了`django.core.mail`这个模块，我们使用它能够很方便的发送邮件。

## 配置SMTP服务器

首先在工程配置`settings.py`中，加入邮件SMTP服务器相关的内容，这里以QQ邮箱为例。

settings.py
```python
EMAIL_HOST = 'smtp.qq.com'
EMAIL_PORT = 465
EMAIL_HOST_USER = 'ciyaz@qq.com'
EMAIL_HOST_PASSWORD = 'qqqqvlaaaaojkkkk'
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
EMAIL_USE_SSL = True
```

注：QQ邮箱需要手动开启SMTP服务接入，并获取密码，上面例子中的密码并不是真实的。

## 发送邮件

这里我们直接编写一个`View`方法，浏览器访问这个方法，就会触发发送邮件的操作。

```python
def mail_test(request):
    mail.send_mail(subject='测试标题',
                   message='测试邮件内容',
                   from_email=settings.EMAIL_HOST_USER,
                   recipient_list=['demo@ciyaz.com'])
    return HttpResponse('OK')
```

其中`send_mail`的几个关键字参数意义如下：

* subject：邮件标题
* message：邮件内容
* from_email：发送人
* recipient_list：接收人

## 发送HTML邮件

上面发送邮件的内容是纯文本的，Django也支持发送HTML的富文本邮件，但这需要借助模板引擎的渲染功能，将我们的数据渲染成HTML，下面代码是一个例子。

我们的模板：
```html
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
<head>
    <meta charset="UTF-8">
    <title>测试表格</title>
</head>
<body>
<table>
    <tr>
        <td>用户名</td>
        <td>密码</td>
    </tr>
    {% for d in data %}
        <tr>
            <td>{{ d.username }}</td>
            <td>{{ d.password }}</td>
        </tr>
    {% endfor %}
</table>
</body>
</html>
```

Python代码：
```python
from django.core import mail
from django.http import HttpResponse
from django.template.loader import render_to_string
from django.utils.html import strip_tags
from demo import settings


def mail_test(request):
    tpl_rendered = render_to_string('mail.html', {
        'data': [
            {'username': 'Tom', 'password': 'abc123'},
            {'username': 'Lucy', 'password': 'hello'},
            {'username': 'Lily', 'password': '54321'}
        ]
    })
    mail.send_mail(subject='账户信息',
                   message=strip_tags(tpl_rendered),
                   html_message=tpl_rendered,
                   from_email=settings.EMAIL_HOST_USER,
                   recipient_list=['demo@ciyaz.com'])
    return HttpResponse('OK')
```

上面代码中，我们使用`render_to_string()`手动调用Django的模板引擎来将数据渲染成模板的HTML内容，然后作为字符串返回，`send_mail()`函数中，我们额外添加了一个关键字参数`html_message`，这个参数包含的内容就是富文本的邮件。但是并不是所有的邮件客户端都支持富文本，`message`参数还是有必要存在的，这里我们使用`strip_tags()`函数将渲染结果中的HTML标签都去掉了，作为纯文本方式的结果。
