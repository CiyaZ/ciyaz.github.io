# Spider 蜘蛛

经过前面的学习，实际上我们已经能够开始使用scrapy完成一些工作了。Spider类我们已经比较熟悉了，这里我们再根据官方文档，详细介绍一下Spider这个类。

## Spider究竟干了什么？

1. 根据`start_urls`定义的初始URL，使用`start_requests()`发起初始请求，发起的请求的方法是返回`Request`类。同时指定一个`parse()`函数，作为请求返回信息处理的回调函数。
2. 回调函数中，解析请求，提取数据，返回的内容可能是这几种：`Item`对象（数据的实体类，后面章节会详细讲），`Request`对象（指定接下来的请求），或使用`yield`语句返回这些对象的生成器。
3. 返回的数据可能直接保存到json文件中，或者调用`Pipeline`持久化到数据库。

## scrapy.Spider

```
class scrapy.spiders.Spider
```

我们编写的自定义Spider，必须继承这个类。`scrapy.Spider`中什么都没做，只是提供了一个默认的函数`start_requests()`，它的作用是根据`start_urls`发起初始请求。scrapy要求我们我们继承这个类的目的，就是把这个类作为模板，使程序结构更加清晰而已。

### 常用属性

下面我们看看Spider类中常用的属性：

* `name` Spider的名字。我们必须在我们的Spider中定义这个属性，scrapy就是通过这个属性区分Spider类的。例如我们爬取的网站域名是`mywebsite.com`，最佳实践是将我们的Spider命名为`mywebsite`。
* `allowed_domains` 这个属性指定了Spider允许爬取的的主机域名。由于我们的爬虫请求的地址可能是从html中动态解析得到的，这个地址可能不在我们想要爬取的主机上，如果不限制爬虫爬取的地址的话，可能会爬到大量无用的数据，这种情况下，我们就可以通过这个属性解决了。
* `start_urls` 爬虫发起初始请求的URL。
* `custom_settings` 这个属性是字典类型，可以包含对scrapy的设置信息，指定这个属性，就会覆盖`settings.py`中的设置的全局定义。

### 常用方法

* `start_requests()` 这个方法我们已经很熟悉了，是用来发起初始请求的。
* `parse(response)` 这个是Request对象的默认回调函数。

## parse(response) 处理逻辑

我们爬虫的处理逻辑都定义在`parse(response)`这个方法中，第一节的例子我们已经介绍过了。

`parse()`中，我们可以`yield`返回一个`Item`对象，代表一个采集到的数据实体。也可以`yield`返回一个`Request`对象，那么Scrapy框架将自动再发起一个新请求。

返回`Request`对象时，有两个经常使用的关键字参数，这里需要介绍下。下面是一个例子：

```python
def parse(self, response):
    # ... 这里省略不相关的代码 ...
    yield response.follow(
        chapter_href,
        meta={
            'nid': response.meta['nid'],
            'chapter_title': chapter_title,
            'chapter_order': self.chapter_order
        },
        callback=self.parse_novel_page)
```

上面代码中，`chapter_href`是链接地址，我们不用关心，主要看`meta`和`callback`这两个关键字参数。

`meta`用于给子请求传递数据，这个实际开发中是肯定会用到的，因为嵌套请求中，父子请求一般是有某种关联的，子请求一般都得通过某种主键一类的标识，知道它的父请求是谁，这种情况就会用到`meta`参数。

另外，Spider组件默认只有个`parse()`处理请求返回内容，如果父子请求的结果都传给`parse()`处理，代码就比较乱了。实际上我们可以再任意定义个方法用于处理子请求，传给`callback`参数。上面代码中，子请求就会由`parse_novel_page()`处理。
