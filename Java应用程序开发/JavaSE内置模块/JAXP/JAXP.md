# XML解析库

从JDK6开始，JDK中包含了Xerces解析库，用来以DOM形式解析XML文档。Xerces由Apache开发，它实现了标准的JAXP（Java API for XML Processing）规范，而且Xerces直接内置在JDK中，使用起来非常方便，性能也不错，Xerces还实现了XPath表达式处理的API，对于查找XML中的特定数据十分方便。除了Xerces，还有Dom4J，JDOM也都是流行的XML解析第三方库。JDK6还引入了现代化的StAX解析器，可以用于XML的SAX解析。

* DOM 以文档树形式读取整份XML文档，全部载入内存，然后由用户解析，速度快，内存占用高，不仅能用来解析XML，还能在内存创建、修改DOM树，写入XML文件
* XPath 通过表达式迅速找到DOM树中的某个数据
* SAX 事件驱动的流式XML解析，速度慢，内存占用低

## DOM解析XML例子

首先我们要了解一些DOM解析规范。整个DOM树结构，被称为“文档”，即`Document`。每一个XML标签节点，每一个文字节点，都被称为“子节点”，即`Node`。XML标签节点比较特殊，它还能包含子节点，因此被称为“元素”，即`Element`，Element接口继承于Node接口。文本节点也比较特殊，它就是一个字符串，因此被称为“文本”，即`TEXT`，TEXT也继承于Node。

像“属性”这种概念，肯定是Element独有的，取字符串值则是TEXT独有的。但是一个Node的子节点既有Element也有TEXT，那么遍历的接口子节点类型全部为Node，我们使用的时候就需要通过`instanceof`进行判断，然后强制类型转换。

```xml
<?xml version="1.0" encoding="utf-8" ?>
<font>
	<name>Source Code Pro</name>
	<size unit="pt">35</size>
</font>
```

```java
import org.w3c.dom.*;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import java.io.File;

class Main
{
	public static void main(String[] args) throws Exception
	{
		DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
		DocumentBuilder builder = factory.newDocumentBuilder();
		File file = new File("test.xml");
		Document document = builder.parse(file);

		//获取根节点
		Element root = document.getDocumentElement();
		//输出节点名字
		System.out.println(root.getTagName());
		//获取子节点
		NodeList children = root.getChildNodes();
		for(int i = 0; i < children.getLength(); i++)
		{
			//根据索引取得子节点
			Node child = children.item(i);
			//只获取子标签节点（排除换行符等被识别为文字节点）
			if(child instanceof Element)
			{
				//读取文字节点的值
				Element childElement = (Element) child;
				Text textNode = (Text) childElement.getFirstChild();
				System.out.println(textNode.getData().trim());

				//读取节点属性值
				if(childElement.getTagName().equals("size"))
				{
					System.out.println(childElement.getAttribute("unit"));
				}
			}
		}
	}
}
```

JAXP接口的内容都很容易理解，上面代码虽然没有列出全部接口的使用方式，但是用到时，看下文档是十分容易的。

## XPath定位XML中数据例子

有关XPath语法请参考`软件开发相关工具/特种用途表达式/XPath表达式`章节。

下面例子在上面代码得到的`document`对象基础上实现。

```java
//获取XPath对象
XPathFactory xPathFactory = XPathFactory.newInstance();
XPath xpath = xPathFactory.newXPath();

//读取文本节点
String sizeValue = xpath.evaluate("/font/size", document);
System.out.println(sizeValue);

//读取节点属性
String sizeAttribute = xpath.evaluate("/font/size/@unit", document);
System.out.println(sizeAttribute);
```
