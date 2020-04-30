# Settings 配置文件

第一篇笔记我们就介绍到，`settings.py`可以用来编写当前scrapy工程的配置。除此之外，Spider中还能通过类属性，指定Spider的私有配置。

这篇笔记我们就具体看一看，我们有如何使用`settings.py`，以及哪些配置可以修改。

## 如何读取配置

Spider中可以通过`self.settings`读取配置信息：

```python
class MySpider(scrapy.Spider):
    name = 'myspider'
    start_urls = ['http://example.com']

    def parse(self, response):
        print("Existing settings: %s" % self.settings.attributes.keys())
```

## 可用配置

由于配置太多了，大部分我都没有写，这里只是写了几个很常用的配置选项。具体所有配置请参考官方文档。

### BOT_NAME

用于指定当前scrapy工程名。这个配置可能被用于`User-Agent`的构建。我们使用scrapy命令行工具创建工程时，这个名字会默认自动指定。

### CONCURRENT_ITEMS

数据处理管线（pipeline）处理数据的最大线程数。默认为16。

### CONCURRENT_REQUESTS_PER_DOMAIN

每一个域名最大的并发请求数。默认为8。

### CONCURRENT_REQUESTS_PER_IP

每个服务器IP的最大并发请求数。如果设置不为0，则`CONCURRENT_REQUESTS_PER_DOMAIN`配置被屏蔽。默认为0。

### DEFAULT_REQUEST_HEADERS

默认请求头。

默认值：
```python
{
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en',
}
```

### DEPTH_LIMIT

爬虫的最大搜索深度。默认为0，表示深度不限。

## Spider局部配置

如果`settings.py`中的全局配置不符合某个Spider的要求，我们可以在Spider中单独进行局部配置，这需要用到`custom_settings`字段，其类型是一个字典，配置项和`settings.py`相同。
