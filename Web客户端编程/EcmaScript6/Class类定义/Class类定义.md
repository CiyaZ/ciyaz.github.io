# 对象扩展

JavaScript的面向对象和继承是通过原型`Prototype`实现的，这和主流语言不同，原型机制设计造成了JavaScript中很多诡异特性（比如JavaScript继承的写法，打个比方：就像Java如果不支持继承，必须得拿反射实现继承一样麻烦），是一个相当糟糕的设计，一般正常人学到这里都会比较困扰。ES6引入了`class`关键字，能够让熟悉主流语言的程序员感到亲切。

## class定义类

实际上，`class`仅仅是ES6引入的一个语法糖，JavaScript面向对象的本质还是原型对象机制。

ES5中通过原型机制模拟一个类（注意是定义一个能实例化的“类”，而不是直接定义一个“对象”）：
```javascript
function Point(x, y) {
    this.x = x;
    this.y = y;
}

Point.prototype.toString = function () {
    return '(' + this.x + ',' + this.y + ')';
};

let p = new Point(1, 2);
console.log(p.toString());
```

ES6中定义一个类：
```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }

    toString() {
        return '(' + this.x + ',' + this.y + ')';
    }
}

let p = new Point(1, 2);
console.log(p.toString());
```

这里我们使用了`class`关键字，实际上结果和ES5的写法是完全相同的，我们定义的`toString()`方法还是定义在`prototype`对象上的。

## 静态方法

我们可以用`static`关键字定义一个静态方法，静态方法不能在类的实例上调用，只能通过类名调用。

```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }

    static createPoint(x, y) {
        return new Point(x, y);
    }

    toString() {
        return '(' + this.x + ',' + this.y + ')';
    }
}
```

## 静态属性

ES6中可以用如下方式定义静态属性：

```javascript
class Point {
    ...//Point类的定义
}
//定义静态属性
Point.i = 1;
```

静态属性调用时也只能通过类名进行调用。

注意：定义静态属性只有上面一种写法，比如试图把`i`写在`class`定义内部之类的写法都不行。这是ES6语法对`class`关键字规定的。

## extends实现类的继承

和主流语言产不多，ES6中使用`extends`关键字实现继承，子类的构造函数中，需要通过`super()`调用父类构造函数。子类能够覆盖父类的同名方法。

```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }

    toString() {
        return '(' + this.x + ',' + this.y + ')';
    }
}

class ColorPoint extends Point{
    constructor(x, y, color) {
        super(x, y);
        this.color = color;
    }

    toString() {
        return '(' + this.x + ',' + this.y + ',' + this.color + ')';
    }
}

let c = new ColorPoint(1, 2, 'RED');
console.log(c.toString());
```
