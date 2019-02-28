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
