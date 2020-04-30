# Items数据实体类

面向对象是软件工程中的最佳实践之一，我们的爬虫爬取的数据，通常需要封装成一个实体类。例如：爬取一个论坛的帖子，那么我们可以设计一个实体类，包含帖子标题，帖子内容，帖子发布时间。使用类对数据进行封装，程序结构就比较清晰，可扩展性和可维护性都比较好。

Spider类可以直接返回Item类，scrapy会自动对其进行处理。

## 定义Item类

```python
import scrapy

class Product(scrapy.Item):
    name = scrapy.Field()
    price = scrapy.Field()
    stock = scrapy.Field()
    last_updated = scrapy.Field(serializer=str)
```

注意：我们继承了`scrapy.Item`类，定义了我们自己的Item类。除此之外，我们定义了类属性（注意类属性的定义，可参考Python/Python面向对象编程 章节），`Field()`实际上是`Field`类的构造方法，`Field`对象可以接受任意类型的数据。

## 使用Item对象

下面我们直接看几个例子，学习一下Item相关的API，我们的`Item`就使用上面定义的Product类。

### 创建items

```python
>>> product = Product(name='Desktop PC', price=1000)
>>> print product
Product(name='Desktop PC', price=1000)
```

### 获取Field值

```python
>>> product['name']
Desktop PC
>>> product.get('name')
Desktop PC

>>> product['price']
1000

>>> product['last_updated']
Traceback (most recent call last):
    ...
KeyError: 'last_updated'

>>> product.get('last_updated', 'not set')
not set

>>> product['lala'] # getting unknown field
Traceback (most recent call last):
    ...
KeyError: 'lala'

>>> product.get('lala', 'unknown field')
'unknown field'

>>> 'name' in product  # is name field populated?
True

>>> 'last_updated' in product  # is last_updated populated?
False

>>> 'last_updated' in product.fields  # is last_updated a declared field?
True

>>> 'lala' in product.fields  # is lala a declared field?
False
```

### 设置Field值

```python
>>> product['last_updated'] = 'today'
>>> product['last_updated']
today

>>> product['lala'] = 'test' # setting unknown field
Traceback (most recent call last):
    ...
KeyError: 'Product does not support field: lala'
```

### 获取键或值集合

```python
>>> product.keys()
['price', 'name']

>>> product.items()
[('price', 1000), ('name', 'Desktop PC')]
```

### 其他常见用法

复制Item
```python
>>> product2 = Product(product)
>>> print product2
Product(name='Desktop PC', price=1000)

>>> product3 = product2.copy()
>>> print product3
Product(name='Desktop PC', price=1000)
```

将Item转换为字典
```python
>>> dict(product) # create a dict from all populated values
{'price': 1000, 'name': 'Desktop PC'}
```

从字典创建Item
```python
>>> Product({'name': 'Laptop PC', 'price': 1500})
Product(price=1500, name='Laptop PC')

>>> Product({'name': 'Laptop PC', 'lala': 1500}) # warning: unknown field in dict
Traceback (most recent call last):
    ...
KeyError: 'Product does not support field: lala'
```

## 关于嵌套Item

如果两个Item实体有关联关系，我们可能会试图创建嵌套的Item，比如`CategoryItem`和`BookItem`，两者就是一对多包含关系。

但据我经验来说，不要这么做！

以目录和书籍为例，数据抓取时，我们并不知道一个目录有几本书，那么这个嵌套对象持久化的时机就是不确定的，内存中未处理完成的对象可能变得过于巨大，一个子对象的异常也可能导致整组数据都处理失败。

另一方面，关联关系其实可以细分成一对一、一对多、多对多，其中又有单双向之分（单向一对多其实是一对多和多对一两种），Scrapy的Item系统并不是一个ORM框架，这个持久化时手动处理起来就很烦了。

比较好的做法还是不要引入实体类的嵌套，就用一个字段来表达这种关联关系即可，也不要强求去约束它。这样采集到一组结果数据后，先存进像MongoDB，或是MySQL的JSON字段这种地方，再另写一个程序，进行后续的数据清洗、转换、处理、入库步骤。
