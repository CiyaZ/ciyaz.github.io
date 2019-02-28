# struts2 通用标签

struts2引入了OGNL表达式，引入了context map和value stack，这些概念都比原本的Servlet+JSP复杂的多。为了兼容这些概念，struts2中引入了专用的标签库，取代JSTL。因此，在struts2项目中，建议使用struts2标签库+OGNL表达式，而不是JSTL+EL表达式。但是struts2标签有个缺点，其属性究竟应该写字符串还是OGNL表达式比较难以分清，这个就得靠经验和文档了（JSTL也有一点类似的问题）。

之前表单相关章节已经介绍了和表单相关的一些标签，这里再介绍一些用于数据展示的通用标签。struts2标签非常多，无法一一介绍，具体可以参考文档。

## 设置变量和输出变量

```xml
<!-- 设置变量 -->
<s:set value="'val1'" var="v1"/>
<!-- 输出值 -->
<s:property value="#v1"/>
```

`<s:set>`的可用属性：

* value：变量的值OGNL表达式
* scope：取值application，session，request，page，action。action表示同时放到request和context map中。
* var：变量名

`<s:property>`的可用属性：

* value：取值用的OGNL表达式
* escapeHtml：是否将输出值的字符进行转义（可用于防止XSS）

## 实例化bean

```xml
<!-- 实例化bean -->
<s:bean name="com.ciyaz.domain.User" var="user">
  <s:param name="username" value="'abc'"></s:param>
  <s:param name="password" value="'123'"></s:param>
</s:bean>
<s:property value="#user.toString()"/>
```

`<s:bean>`会实例化类，放入context path中。构造函数参数通过`<s:param>`传入，注意其`value`属性是OGNL表达式。

* name：类全名
* var：变量名

## 循环

```xml
<!-- 循环List -->
<s:set value="{'a','b','c'}" var="list" />
<s:iterator value="#list" var="l">
  <s:property value="#l" />
</s:iterator>
<!-- 循环Map-->
<s:set value="#{'1':'a','2':'b','3':'c'}" var="map" />
<s:iterator value="#map" var="m">
  <s:property value="#m.key" />
  <s:property value="#m.value" />
</s:iterator>
```

* value：循环的List或Map对象
* var：循环变量

除了上面代码中的属性，`<s:iterator>`还有一个`status`属性，可以设置一个状态对象，该状态对象包含以下属性：

* status.isOdd
* status.isEven
* status.isLast
* status.isFirst
* status.getIndex
* status.getCount

## 判断

```xml
<!-- 判断 -->
<s:set value="-1" var="v2" />
<s:if test="#v2 == -1">
  negative
</s:if>
<s:elseif test="#v2 == 0">
  zero
</s:elseif>
<s:else>
  positive
</s:else>
```

* test：条件表达式

## URL定义和链接输出

```xml
<!-- URL -->
<s:url action="upload" var="u1">
  <s:param name="name" value="'abc'"></s:param>
</s:url>
<s:a href="%{#u1}">click here</s:a>
```

`<s:url>`的属性：

* action：动作名
* var：存储到的变量名

`<s:a>`的属性：

* href：链接地址，默认是字符串，想使用变量需要用`%{}`包裹

链接输出结果
```
http://localhost:8080/struts2_demo2/upload.action?name=abc
```

可以发现，`<s:url>`帮我们自动添加了动作后缀。
