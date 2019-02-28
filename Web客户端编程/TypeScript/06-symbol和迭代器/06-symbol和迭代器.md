# 迭代器

## symbol
一些内置于对象的Symbol属性，用于表示对象的语言行为。

以下为这些symbols的列表：

### Symbol.hasInstance

方法，会被instanceof运算符调用。构造器对象用来识别一个对象是否是其实例。

### Symbol.isConcatSpreadable

布尔值，表示当在一个对象上调用Array.prototype.concat时，这个对象的数组元素是否可展开。

### Symbol.iterator

方法，被for-of语句调用。返回对象的默认迭代器。

### Symbol.match

方法，被String.prototype.match调用。正则表达式用来匹配字符串。

### Symbol.replace

方法，被String.prototype.replace调用。正则表达式用来替换字符串中匹配的子串。

### Symbol.search

方法，被String.prototype.search调用。正则表达式返回被匹配部分在字符串中的索引。

### Symbol.species

函数值，为一个构造函数。用来创建派生对象。

### Symbol.split

方法，被String.prototype.split调用。正则表达式来用分割字符串。

### Symbol.toPrimitive

方法，被ToPrimitive抽象操作调用。把对象转换为相应的原始值。

### Symbol.toStringTag

方法，被内置方法Object.prototype.toString调用。返回创建对象时默认的字符串描述。

### Symbol.unscopables

对象，它自己拥有的属性会被with作用域排除在外。

## 可迭代性

实现了Symbol.iterator属性，我们认为它是可迭代的。一些内置类型如：Array，Map，Set，String，Int32Array，Uint32Array都实现了它。对象上的Symbol.iterator函数负责返回供迭代的值。

## for...of...和for...in...

一个迭代键，一个迭代值：

```javascript
let list = [4, 5, 6];
for (let i in list) {
    console.log(i); // "0", "1", "2",
}
for (let i of list) {
    console.log(i); // "4", "5", "6"
}
```
