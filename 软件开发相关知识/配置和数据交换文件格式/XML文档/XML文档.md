# XML文档

XML（Extensible Markup Language），中文全称为可标记扩展语言。XML是一个非常有用的描述结构化信息的技术，可以作为数据交换格式，配置文件等。XML尤其和Java配合的非常好，JavaWeb项目，Android项目中总是会碰到XML。

Java中，除了XML还可以使用属性文件`.properties`，但是XML能够表示层次结构，甚至可以直接映射到Java类，这些都是属性文件做不到的。XML的层次结构语法，不仅非常直观，而且易于计算机解析。唯一的缺点就是字符实在太多，在进行网络上的数据交换时，效率不高（因此出现了json）。

## XML文档的结构

XML以一个文档头开始：

```xml
<?xml version="1.0" encoding="utf-8" ?>
```

* version：XML一共两个版本，1.0和1.1，XML1.1的新特性几乎不会用到，因此填写1.0即可
* encoding：文件的编码，统一使用utf-8即可

文档头下面通常是dtd或Schema约束的声明。约束文件的作用就是规定XML如何编写，该有哪些节点，该有哪些属性等。约束声明是可选的，如果只是自己的项目用XML数据交换，没有约束声明的XML也能正常工作。

一个Hibernate框架配置文件的dtd约束声明
```xml
<!DOCTYPE hibernate-configuration PUBLIC
		"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
		"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd" >
```

dtd或schema也有自己的语法，编写约束文件是一件很繁琐的事。但是各种框架总是不厌其烦的编写，其原因之一就是dtd或schema载入IDE后（eclipse或Intellij IDEA）等，IDE会给编写XML的程序员合理的提示，否则程序员连要写什么属性都不知道，配置文件写错一个字母，程序崩溃，问题也难以发现。除此之外，XML解析器也可以利用约束文件，帮助XML解析。

最后就是XML的正文了。正文必须以一个根元素开始，根元素包含子元素，形成树形结构，所有节点都可以拥有属性，叶子节点还可以包含文本。

```xml
<fonts>
	<font>
		<name>Consolas</name>
		<size unit="pt">36</size>
	</font>
	<font>
		<name>Source Code Pro</name>
		<size unit="pt">35</size>
	</font>
</fonts>
```

关于何时作为文本节点，何时作为属性，有一些争议。比如上面的`<size unit="pt">36</size>`，写成`<size unit="pt" n="36" />`是否可以呢？这样设计是不好的。我们可以类比HTML，HTML中文本节点都是显示到网页上的数据，属性都是用来控制标签如何显示文本节点的（先不管submit的value这种异类）。XML也是同理，真正的数据值就该放到文本子节点，属性则是用来修饰数据值的。

## 一些其它的标记

* 字符引用：非ASCII字符可以使用`&#十进制值`或`&#x十六进制值`的形式表示

* 实体引用：一些和XML语法冲突的字符，如`<`可以表示为`&lt;`

* CDATA：使用`<![CDATA[cdata字符串]]>`包裹的内容，其中的特殊字符不需要使用实体引用标记

* 注释：`<!--这是注释-->`注意，注释中不能含有`--`，否则会造成解析错误
