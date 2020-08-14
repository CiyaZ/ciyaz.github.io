# momentjs-日期处理

JavaScript自带的日期时间处理标准库，功能实际上并不是很友好，`momentjs`是一个比较强大的日期时间处理库，能够解析、格式化、检验、计算日期和时间。它比较接近Java的日期API（Date和Calendar），熟悉Java的同学应该能很快掌握。

中文文档：[http://momentjs.cn/docs/#/](http://momentjs.cn/docs/#/)
Github地址：[https://github.com/moment/moment](https://github.com/moment/moment)

## 格式化日期时间

下面例子中，我们能够输出形如`2020-08-14 17:49:19`这种格式的时间：

```javascript
var now = moment();
var nowStr = now.format('YYYY-MM-DD HH:mm:ss');

console.log(nowStr);
```

`moment()`如果不传入任何参数，其默认会初始化当前时间（通过浏览器取得）。

格式化符号具体可以参考文档。

## 解析日期时间

`moment.js`支持多种时间格式，比较常用的是'YYYY-MM-DD HH:mm:ss'格式：

```javascript
var now = moment('2020-08-14 17:49:19');
var nowStr = now.format('YYYY-MM-D HH:mm:ss');

console.log(nowStr);
```

## 日期时间取值和赋值

`moment.js`封装了一些取值函数，可以单独取出当前年、月等字段：

```javascript
var now = moment('2020-08-14 17:49:19');
console.log('年', now.year());
console.log('月', now.month());
console.log('日', now.date());
console.log('星期', now.day());
```

我们也可以用同样的方式来赋值：

```javascript
now.date(1);
```

注意：

1. 月的值是从`0`开始的。
2. 赋值时如果溢出，会自动向后推算

## 日期时间的加减计算

下面代码将日期向后增加了31天。

```javascript
var now = moment('2020-08-14 17:49:19');
now.add(31, 'd');
```

其中，`add()`的参数`d`表示增加的是天这个单位，它是一个简写，我们也可以传字符串`days`。如果计算后有溢出，会自动向后推算。

下表是单位参数：

```
years	        y
quarters	    Q
months  	    M
weeks	        w
days	        d
hours	        h
minutes	        m
seconds	        s
milliseconds	ms
```

另一个常见的需求是求两个日期的相差天数，这可以用`diff()`函数实现：

```javascript
var datetime1 = moment('2020-08-14 17:49:19');
var datetime2 = moment('2020-08-15 17:49:19');

var diff = datetime1.diff(datetime2, 'd');
console.log(diff);
```

注意：以比较天数为例，如果指定了时间，那么时间也会被考虑进去，例如：'2020-08-14 17:49:19'和'2020-08-15 17:49:18'相差的天数为0，因为差了一秒。
