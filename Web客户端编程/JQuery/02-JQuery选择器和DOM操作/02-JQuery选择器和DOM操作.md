# JQuery选择器

JQuery选择的基本用法是：`$(selector).action()`，美元符号是JQuery对象的别名，selector是选择符字符串，action()是执行对元素的操作。

文档就绪函数（作用相当于javascript的window.onload）：

```javascript
//文档加载完成时回调
$(document).ready(function(){
});
```

## html选择器

JQuery使用css选择器的语法选择html元素，使用xpath表达式语法选择带有给定属性的html元素。

常用选择器：
```javascript
$("#id")                                //ID选择
$("div")                                //元素选择
$(".classname")                         //类选择
$(".classname,.classname1,#id1")        //组合选择

$("#id>.classname")                     //子元素选择

$("li:first")                           //第一个li
$("li:last")                            //最后一个li
$("li:even")                            //挑选下标为偶数的li
$("li:odd")                             //挑选下标为奇数的li
$("li:eq(4)")                           //下标等于4的li
$("li:gt(2)")                           //下表大于2的li
$("li:lt(2)")                           //下标小于2的li

$("div:contains('aa')")                 // 包含文本aa的元素
$("td:empty")                           //不包含子元素或者文本的空元素
$("div:has(selector)")                  //含有选择器所匹配的元素
$("td:parent")                          //含有子元素或者文本的元素

$("div[id]")                            //所有含有 id 属性的 div 元素
$("div[id='123']")                      // id属性值为123的div 元素
$("div[id!='123']")                     // id属性值不等于123的div 元素
$("div[id^='qq']")                      // id属性值以qq开头的div 元素
$("div[id$='zz']")                      // id属性值以zz结尾的div 元素
$("div[id*='bb']")                      // id属性值包含bb的div 元素
$("input[id][name$='man']")             //多属性选过滤，同时满足两个属性的条件的元素
```

# DOM操作

## JQuery对象和DOM对象

JQuery选择得到的是DOM对象，JQuery对象可以看做有JQuery特殊功能的DOM对象的数组，因为JQuery对象都可以隐式迭代，从JQuery对象获得DOM对象也和数组很像：`var div = $("#d1")[0]`，DOM对象转JQuery对象则是这样：`var jq_div = $(div)`。下面介绍JQuery对象特有的功能。

## 获取内容

* text()：读取/设置元素的文本内容
* html()：读取/设置元素的内容，包括html标记
* val()：读取/设置表单字段的值

## 获取属性

* attr()：读取/设置属性

## 修改css属性

* css()：读取/设置css样式，例如：$("#d1").css("background-color", "red");
* addClass()：添加class属性
* removeClass()：删除class属性

## 添加元素

* append()：在被选元素结尾插入子元素
* preappend()：在被选元素开头插入子元素
* after()：在被选元素之前插入
* before()：在被选元素之后插入

接收参数：可以是文本，这样就插入文本节点，也可以是Element，也可以是JQuery对象

## 删除元素/内容

* remove()：删除被选元素及其子元素，也可以接受一个符合JQuery语法的字符串参数，用于指定删除哪些元素
* empty()：删除被选元素的子元素

## 尺寸处理

* width()：读取/设置元素宽度（不包括内边距，边框，外边距）
* height()：读取/设置元素高度（不包括内边距，边框，外边距）
* innerWidth()：读取/设置元素包括内边距宽度
* innerHeight()：读取/设置元素包括内边距高度
* outerWidth()：读取/设置元素包括内边距和边框宽度
* outerHeight()：读取/设置元素包括内边距和边框高度

## 遍历

### 向父节点遍历

* parent()返回上一级父元素
* parents()返回所有祖先元素
* parentsUntil()参数表示向父节点遍历直到（不包括终止条件），返回所有符合结果的元素

### 向子节点遍历

* children()返回向下一级的子元素，接受选择器参数
* find()返回所有子元素，接受选择器参数

### 同胞遍历

* siblings()返回所有同胞，接受选择器参数
* next()返回被选元素的下一个同胞
* nextAll()返回后续所有同胞
* nextUtil()后续同胞，接受终止条件
* prev() prevAll() prevUtil()同上

## 元素集合操作

上面已经说过JQuery对象和DOM对象的关系，JQuery对象默认就是类似于数组的，可能包含不止一个html元素。

* first()：返回第一个对象
* last()：返回最后一个对象
* eq()：参数为索引值，注意和直接下标取的区别：这里取的是JQuery对象，直接下标取是DOM对象
* filter()：参数为选择器，选择符合条件的对象
