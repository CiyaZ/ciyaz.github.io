# Item Pipeline 数据处理管线

上一篇笔记我们介绍了Item的用法，实际上Item就是对数据的封装，Spider中请求对象的回调函数可以返回Item类型，scrapy会自动对Item进行处理。那么，scrapy根据什么处理Item对象呢？实际上这就是Item处理管线（Pipeline）的工作了。Item基本都是和Pipeline一起工作的。

管线（pipeline）：在计算机科学中，用来描述一种处理过程，例如OpenGL的渲染管线，数据库实现中的查询执行管线等。注意不要和术语`pipe`弄混了，pipe中文译为管道，通常指操作系统的一种进程间通信机制。

## pipeline的作用

scrapy中pipeline的用途：

1. 整理HTML字符串数据
2. 检查爬取到的数据，看看它是否是我们想要的
3. 查重，决定我们是否丢弃这个Item
4. 持久化到数据库

## 如何编写我们自己的pipeline

一个简单的pipeline例子：
```python
from scrapy.exceptions import DropItem

class PricePipeline(object):

    vat_factor = 1.15

    def process_item(self, item, spider):
        if item['price']:
            if item['price_excludes_vat']:
                item['price'] = item['price'] * self.vat_factor
            return item
        else:
            raise DropItem("Missing price in %s" % item)
```

每个pipeline都必须实现这个方法：

```python
process_item(self, item, spider)
```

这个方法就是用来对数据进行处理的，我们对数据处理完成后，就通过`return`将其返回。如果我们想丢弃这个数据，就`raise`一个`DropItem`异常（如上面例子）。

除此之外，我们还可以实现这些方法：

* `open_spider(self, spider)` Spider开启后，这个方法会被回调。
* `close_spider(self, spider)` Spider关闭时回调。

这两个函数的作用非常明显，就是打开和关闭数据库。我们总不能每次向数据库写数据时，都重新建立连接吧。

## 配置pipeline

编写完pipeline之后，一定不要忘了在`settings.py`中进行配置。

例子：
```python
ITEM_PIPELINES = {
    'myproject.pipelines.PricePipeline': 300,
    'myproject.pipelines.JsonWriterPipeline': 800,
}
```

后面的数字是指定pipeline的运行顺序，取值0-1000，数字越小优先级越高。

## pipeline使用例子

下面是官网介绍的一些pipeline例子。

### 将数据写入JSON文件

```python
import json

class JsonWriterPipeline(object):

    def open_spider(self, spider):
        self.file = open('items.jl', 'w')

    def close_spider(self, spider):
        self.file.close()

    def process_item(self, item, spider):
        line = json.dumps(dict(item)) + "\n"
        self.file.write(line)
        return item
```

### 将数据写入MongoDB

```python
import pymongo

class MongoPipeline(object):

    collection_name = 'scrapy_items'

    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        self.db[self.collection_name].insert(dict(item))
        return item
```

注：这里我们使用了`pymongo`。
