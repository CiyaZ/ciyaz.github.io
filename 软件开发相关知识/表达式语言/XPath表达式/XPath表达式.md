# XPath表达式

XPath是一个在XML文档中查找信息的语言，它基于XML文档的树形结构进行查找。XPath在提取XML或是HTML文档中信息时非常有用。

## XPath的使用场景

XPath常用于XML配置文件解析，XML网络服务数据解析，网络爬虫HTML文档解析等。

## XPath和正则表达式的区别

正则表达式是基于字符串的模式匹配算法，在目标字符串中匹配模式字符串。XPath则是依据XML的树结构进行解析。两者都能实现从一个XML文档中提取信息，但原理不同，对于解析XML我们应该使用XPath，对于从一篇文章中匹配关键字，显然应该使用正则表达式。

# XML文档中的基本概念

学习XPath语法前，需要了解XML文档中的基本概念。

这是一个标准的XML文档
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<bookstore>

<book>
  <title lang="en">Harry Potter</title>
  <author>J K. Rowling</author>
  <year>2005</year>
  <price>29.99</price>
</book>

</bookstore>
```

## Node 节点

* `<bookstore>` 文档节点
* `<author>J K. Rowling</author>` 元素节点
* `lang="en"` 属性节点
* `J K. Rowling` 基本值，或称原子值，无父或无子的节点

## 节点之间的关系

* Parent 父
* Children 子
* Sibling 同胞
* Ancestor 先辈
* Descendant 后代

# XPath语法

## 使用XPath的编程环境

我们这里使用Python的`lxml`解析库提供XPath支持进行测试。除了Python外，XPath在大多数编程语言上都能使用，我们就不一一说明了。

Python使用XPath示例：
```python
import lxml.etree as et

root = et.fromstring(xml)
for node in root.xpath("/bookstore/book/title"):
	print(node.text)
```

## 路径表达式

* `nodename` 选取此节点的所有子节点
* `/` 从根节点选取
* `//` 从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置
* `.` 选取当前节点
* `..` 选取当前节点的父节点
* `@` 选取属性

## 谓语

谓语使用方括号，表示查找符合某个特定条件的节点。

例如：`//title[@lang='eng']`，选取所有`lang`属性为`eng`的`title`元素。

## 通配符

* `*` 匹配任何元素节点
* `@*` 匹配任何属性节点
* `node()` 匹配任何节点

## 或运算

* `|` 匹配多个路径

## 运算符

```
| + - * div = != < <= > >= or and mod
```

# XPath实例

这一部分介绍使用XPath的实际例子，例子是最有用的。

XML文档：
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<bookstore>

<book category="COOKING">
  <title lang="en">Everyday Italian</title>
  <author>Giada De Laurentiis</author>
  <year>2005</year>
  <price>30.00</price>
</book>

<book category="CHILDREN">
  <title lang="en">Harry Potter</title>
  <author>J K. Rowling</author>
  <year>2005</year>
  <price>29.99</price>
</book>

<book category="WEB">
  <title lang="en">XQuery Kick Start</title>
  <author>James McGovern</author>
  <author>Per Bothner</author>
  <author>Kurt Cagle</author>
  <author>James Linn</author>
  <author>Vaidyanathan Nagarajan</author>
  <year>2003</year>
  <price>49.99</price>
</book>

<book category="WEB">
  <title lang="en">Learning XML</title>
  <author>Erik T. Ray</author>
  <year>2003</year>
  <price>39.95</price>
</book>

</bookstore>
```

## 输出所有书的标题

```python
root = et.fromstring(xml)
for node in root.xpath("/bookstore/book/title"):
	print(node.text)
```

```
Everyday Italian
Harry Potter
XQuery Kick Start
Learning XML
```

## 输出第一个书的标题

```python
root = et.fromstring(xml)
for node in root.xpath("/bookstore/book[1]/title"):
	print(node.text)
```

```
Everyday Italian
```

## 选取所有价格高于35的书名

```python
root = et.fromstring(xml)
for node in root.xpath("/bookstore/book[price>35]/title"):
	print(node.text)
```

```
XQuery Kick Start
Learning XML
```

# 使用lxml解析HTML文档

XML是十分严谨的，而HTML解析则要求很高的容错性，lxml具有这种特性。

我们使用的网页来自这里：[http://quotes.toscrape.com/](http://quotes.toscrape.com/)

## 例子1

```python
import lxml.html

fp = open("a.html", "r")
xml = fp.read()
fp.close()

s = lxml.html.fromstring(xml)
nodes = s.xpath("//a[@class='tag']/@href")
for n in nodes:
	print(n)
```

上面代码解析所有`class`属性为`tag`的`<a>`的`href`属性。

部分结果（实际结果很长）：
```
/tag/change/page/1/
/tag/deep-thoughts/page/1/
/tag/thinking/page/1/
/tag/world/page/1/
...
```

## 例子2

```python
import lxml.html

fp = open("a.html", "r")
xml = fp.read()
fp.close()

s = lxml.html.fromstring(xml)
nodes = s.xpath("//li[@class='next']/a/@href")
for n in nodes:
	print(n)
```

上面代码截取翻页按钮的地址。翻页按钮的特征是这样的：

```html
<li class="next">
    <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
</li>
```

程序输出：
```
/page/2/
```

# 总结

到现在为止，我们已经比较熟悉XPath了，不得不说在爬虫程序中，XPath往往远比正则表达式好写。
