# JSTL标签库

JSTL（JavaServer Pages Standard Tag Library），是JSP的标准标签库。上一篇笔记介绍了JSP+Servlet+JavaBean实现MVC模式的表现层架构，JSP作为模板，需要实现判断、循环等功能，用于HTML输出。JSTL标签就能够实现。当然，JSP最初的目的不仅仅是作为表现层MVC的模板，而是还能够实现Model1模式（JSP+JavaBean）的开发，因此JSTL里也有一些额外的标签，这里不做介绍。

除了JSP之外，实际上还有其他的表现层模板引擎，如`Velocity`等，Velocity使用了自己的VTL语法，而不是类似JSP标签的东西，因此Velocity更像是纯粹的模板引擎，JSP倒像是一个更复杂的技术被用户当成模板引擎使用。Velocity可以在其他章节中找到相关介绍。

## 引入JSTL依赖

Maven中需要配置JSTL的依赖jar包，否则是不能使用JSTL标签库的。

```xml
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>jstl</artifactId>
  <version>1.2</version>
</dependency>
```

## 核心标签

jstl中最常用的就是核心标签，其余很少使用。核心标签也就是能够让JSP作为模板引擎使用的那部分标签。

### JSP中引入核心标签

```html
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```

### out 显示数据

```html
<c:out value="<string>" default="<string>" escapeXml="<true|false>"/>
```

* value 输出内容
* default 输出默认值
* escapeXml 是否忽略xml特殊字符，默认true

例子

```html
<c:out value="${user.username}" />
```

### set 保存数据

```html
<c:set
   var="<string>"
   value="<string>"
   target="<string>"
   property="<string>"
   scope="<string>"/>
```

* value 要存储的值
* target 要修改的属性所属的对象
* property 要修改的属性
* var 存储信息的变量
* scope var属性所属的作用域，默认为page

例子

```html
<c:set value="hello" var="str" />
```

### remove 删除数据

```html
<c:remove var="<string>" scope="<string>"/>
```

* var 要移除的变量名称
* scope 所属作用域，默认是所有作用域

### catch 处理异常

```html
<c:catch var="<string>">
...
</c:catch>
```

* var 用来存储错误信息的变量

### if 条件分支

```html
<c:if test="<boolean>" var="<string>" scope="<string>">
   ...
</c:if>
```

* test 为true显示主体内容
* var 存储条件结果的变量
* scope var的作用域，默认为page

例子
```html
<c:if test="${sessionScope.session_bean != null}">
  <li><a href="http://www.ciyaz.com">博客</a></li>
  <li><a href="${pageContext.request.contextPath}/logout">退出系统</a> </li>
</c:if>
```

### choose条件分支

类似switch...case...，由于JSP核心标签里没有if...else...，因此choose比if更加常用。

```html
<c:choose>
    <c:when test="<boolean>"/>
        ...
    </c:when>
    <c:when test="<boolean>"/>
        ...
    </c:when>
    ...
    ...
    <c:otherwise>
        ...
    </c:otherwise>
</c:choose>
```

* test 条件，为true显示主题内容

### forEach迭代

```html
<c:forEach
    items="<object>"
    begin="<int>"
    end="<int>"
    step="<int>"
    var="<string>"
    varStatus="<string>">
    ...
</c:forEach>
```

* items 被迭代的变量
* begin 起始索引
* end 终止索引
* step 迭代步长
* var 当前条目的变量名称
* varStatus 循环的状态变量

例子

```html
<table border="1">
	<c:forEach items="${collection}" var="v">
		<tr>
			<td><c:out value="${v}" /></td>
		</tr>
	</c:forEach>
</table>
```

# EL表达式

el表达式可以访问jsp中的变量，格式为${}，提供.和[]运算符，例如：${user.username}，${user[2]}，[]内可以是一个变量，用于动态取值。

## 变量作用域

如果el表达式中不指定变量的作用域，会依次从page、request、session、application中查找该变量，也可以用pageScope、requestScope、sessionScope、applicationScope明确指定变量的作用域，例：${requestScope.user.username}。

## 隐式对象

### cookie

el表达式可以直接读取cookie。

例子：获得cookie对象，cookie键，cookie值

```html
<c:out value="${cookie.username}" />
<c:out value="${cookie.username.name}" />
<c:out value="${cookie.username.value}" />
```

### header

el表达式可以读取请求头信息。如果一个键对应多个值，可以用headerValues。

例子
```html
<c:out value="${header['User-Agent']}" />
```

### initParam

相当于application.getInitParam()，不常用。

### pageContext

取得一些页面的其他信息。

例子：

* `${pageContext.request.queryString}` 取得请求的参数字符串
* `${pageContext.request.requestURL}` 取得请求的URL，但不包括请求之参数字符串
* `${pageContext.request.contextPath}` 服务的web application 的名称
* `${pageContext.request.method}` 取得HTTP 的方法(GET、POST)
* `${pageContext.request.protocol}` 取得使用的协议(HTTP/1.1、HTTP/1.0)
* `${pageContext.request.remoteUser}` 取得用户名称
* `${pageContext.request.remoteAddr}` 取得用户的IP 地址
* `${pageContext.session.new}` 判断session 是否为新的
* `${pageContext.session.id}` 取得session 的ID
* `${pageContext.servletContext.serverInfo}` 取得主机端的服务信息

## 表达式运算符

数学运算符：`+ - * /或div %或mod`

关系运算符：`==或eq ！=或ne <或lt >或gt <=或le >=或ge`

逻辑运算符：`&&或and ||或or ！或not`

其他：`empty运算符 三目运算符:?`
