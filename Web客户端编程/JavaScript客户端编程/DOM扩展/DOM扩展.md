# DOM扩展

起初浏览器都有自己的专有DOM扩展，为了统一标准，w3c将一些已经成为事实标准的DOM扩展写进标准内。尽管如此，还是有一些浏览器的专有扩展存在。

## Selector API

这组API描述根据CSS选择符选取与某个模式匹配的DOM元素，实际上JQuery就包含这个功能。Selector API则是w3c指定的一组标准，其两个核心方法是：querySelector()和querySelectorAll()，可以通过Document或Element类型的实例调用他们。

### querySelector()

例子：
```html
<body>
	<div id="d1">
		<p class="c1">hello1</p>
		<p class="c1">hello2</p>
		<p class="c2">hello3</p>
		<p class="c2">hello4</p>
	</div>
	<script>
		//取得id为d1的元素
		var d = document.querySelector("#d1");
		//取得class为c2的第一个元素
		var p = document.querySelector(".c2");
	</script>
</body>
```

如果参数错误，函数会抛出错误。

### querySelectorAll()

返回的是所有匹配的元素，返回值是NodeList类型。用法基本同上。

### matchesSelector()

判断元素是否和CSS选择符匹配，返回true或false。

## Element Traversal API

这组API规范了element元素的遍历。为DOM元素添加了如下属性：

* childElementCount 子元素个数，不包括文本节点和注释
* firstElementChild 指向第一个子元素，不包括文本节点和注释（和firstChild区别就在于此）
* lastElementChild 指向最后一个子元素
* previousElementSibling 指向前一个同辈
* nextElementSibling 指向后一个元素同辈

## HTML5新增API

### 和class相关

#### getElementsByClassName()

获得某个class的所有元素，返回值是NodeList，实际上和Selector API功能有重叠。

#### classList属性

一个元素的class可能是多个，直接获取class属性得到的是包含多个class，以空格分割的字符串，使用classList属性，就能得到class的数组（实际上是DOMTokenList，和数组不同）。

### 焦点管理

#### document.activeElement

这个属性是当前获得焦点的元素的引用。

#### document.hasFocus()

判断当前文档是否获得焦点。

### 新功能扩展

document.readyState 判断文档状态

* loading 正在加载
* complete 加载完成

#### 自定义数据属性

html5中可以为元素添加自定义属性，但是必须以"data-"前缀开头，如：`<div data-test="testValue"></div>`

访问自定义属性例子：

```html
<body>
	<div data-test_attr="testValue" id="d1"></div>
	<script>
		var div = document.querySelector("#d1");
		var value = div.dataset.test_attr;
		console.log(value);
	</script>
</body>
```

注意：不要在自定义属性中出现大写字母，因为`div.dataset`中，只有小写字母属性，但可以用下划线。

#### innerHTML/outerHTML

innerHTML可以读取/修改某个元素内部的html代码，outerHTML可以读取/修改包括该元素在内的html代码。

#### insertAdjacentHTML()

接收两个参数，第一个参数是插入位置字符串，第二个参数是html代码字符串。

插入位置取值：

* beforebegin 在当前元素之前插入同辈元素
* afterbegin 在当前元素第一个子元素前插入
* beforeend 以此类推
* afterend 以此类推
