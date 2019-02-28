# SOAP协议和JAXWS

什么是Web Service？

说白了就是通过HTTP协议或是其他协议在网络上发布的公共接口，最常见的就是Android开发中，经常调一些天气接口之类，传个json串，然后解析下响应，天气接口网站所提供的这种服务就可以叫做“Web Service”。我们直接用Servlet或是SpringMVC可以直接基于HTTP协议开发简单的Web Service，也没什么难度，大多数网络接口都是这么做的。但是，如果业务一旦复杂起来，自定义接口就越来越困难，Java最喜欢搞这些问题，于是弄出了JAX-WS规范，出现了CXF等框架。

基于SOAP的Web Service有三个要素，WSDL，SOAP，UDDI。WSDL用于描述如何访问具体的接口，SOAP描述了具体的信息传输格式，UDDI用来管理、分发和查询Web Service（不常用）。

SOAP协议即简单对象访问协议，它是一种基于XML在HTTP之上封装的应用层协议，Web Service可以基于SOAP协议开发，对服务端开发者和客户端开发者来说，双方都遵循SOAP规范，使用起来就非常方便。SOAP协议由w3c维护。

JAX-WS（全称Java API for XML Web Service），是基于XML的Web Service的Java API规范。JDK6内置的`javax.jws`包对这套API进行了实现，JavaSE中可以直接使用。除此之外，还有许多开源框架实现了JAX-WS，如CXF等。JAX-WS中，一个远程调用可以转化为SOAP协议，和服务器通信，但是开发者不需要关心具体的SOAP协议实现，JDK6以上版本提供了`wsimport`工具，可以直接根据WSDL生成客户端的远程调用代码。

有关SOAP协议和WSDL文档的格式，具体内容可以参考文档，一般不需要程序员关心，这里就不详细介绍了。后面章节代码中会有涉及。

## Web Service的使用场景

这里说的使用场景指基于SOAP协议的Web Service，而不是简单的天气接口等。Web Service的一个比较常见的用途就是用于分布式应用的集成，比如许多学校或企业内部系统有单点登录功能，但是这一大堆系统开发语言都不同，部署环境也不同，如何实现单点登录呢？这里就可以使用Web Service实现。使用SOAP能够实现系统之间的松耦合，对于软件的可扩展性和可维护性有重要意义。

对于同构的程序，比如几个Java开发的系统需要协作，这时使用RMI是更好的选择。RMI使用socket传输，且同样支持面向对象，比SOAP快得多。对于性能要求很高，客户端和服务器需要快速响应的系统，SOAP也是不行的，因为HTTP本来就很慢，上面又套了层SOAP，速度更是慢，要求性能的远程调用或远程通信最好直接在TCP/UDP上封装自己的协议。
