# let/const关键字和块级作用域

在ES6之前，JavaScript中声明一个变量会使用`var`关键字：

```javascript
var a = 1;
```

用`var`声明的变量具有“变量提升”这个“特性”，导致JavaScript的变量看起来比较诡异。比如：

```javascript
function foo(condition)
{
	if(condition)
	{
		var a = 10;
	}
	else
	{
    //在这里访问变量a，得到的是undefined，而不是变量未定义错误
		console.log(a);
	}
}
```

按正常人的审美，上面代码运行到`console.log(a);`应该报个错：变量未定义。而JavaScript偏偏不这样设计，上面代码中`var`定义的变量，对于思维正常的人来说应该这样理解：

```javascript
function foo(condition)
{
  var a;
	if(condition)
	{
		a = 10;
	}
	else
	{
    //在这里访问变量a，得到的是undefined，而不是变量未定义错误
		console.log(a);
	}
}
```

## 用let声明变量

ES6标准推荐使用`let`取代`var`声明变量，具体的改进就不详细说了，总而言之就是`let`声明的变量没有`var`那种诡异的“变量提升”特性，适用于思维正常的人。

`let`声明的变量具有的特性：

* 有传统类C语言的块级作用域
* 不能重复声明
* 不存在覆盖全局对象属性的问题，比如在全局作用于下，用`var`声明的变量会覆盖`window`对象的属性

## 用const声明常量

使用`const`可以声明常量，常量和`let`声明的变量具有相同的块级作用域。

* `const`声明的常量必须在声明时赋值
* 常量不能重新赋值，如果常量是对象的引用，对象的属性值可以随意改
