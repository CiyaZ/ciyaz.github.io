# Symbol类型

在ES6中引入了一个新的基本数据类型`Symbol`，本片笔记记录Symbol的用法。

## Symbol的意义

没有Symbol时，我们创建的JavaScript对象中会带有大量的键值对属性，例如下面代码：

```javascript
let user = {
    userType: '辣鸡用户',
    username: 'Tom',
    age: 8
};

console.log(user);
```

如果我们只是想标识一个对象的某种“类型”，而不想让这种类型作为对象属性，引入Symbol就能解决这个问题：

```javascript
let userType = Symbol('userType');

let user = {
    [userType]: '辣鸡用户',
    username: 'Tom',
    age: 8
};

// 访问Symbol属性
console.log(user[userType]);

// JSON序列化、循环等无法得到Symbol属性
let userJson = JSON.stringify(user);
console.log(userJson);
```

## Symbol的特点

任何两个Symbol对象都是不同的：

```javascript
console.log(Symbol() === Symbol());
```

输出：
```
false
```

因此，一次实例化创建的Symbol是一个唯一的标识。我们可以通过`Symbol.for()`获取一个已经注册过的标识：

```javascript
// Symbol对象不存在会创建
let a = Symbol.for('myType');
// Symbol对象存在，直接获取
let b = Symbol.for('myType');
console.log(a === b);
```

输出：
```
true
```

## Symbol的用途

实际上，我们自己定义Symbol的状况还是比较少的，毕竟多引入一个概念会让代码变得更加复杂，大多数需求普通属性都能实现。

我们一般用到Symbol的状况是在对象上，赋予JavaScript内置的一些Symbol，以实现某种功能。例如，让一个自定义对象变得可迭代（这里是指使用ES6的`for...of...`）。

```javascript
let myObj = {
    0: 'a',
    1: 'b',
    2: 'c',
    length: 3,
    [Symbol.iterator]: function () {
        let p = 0;
        return {
            next: () => {
                return {
                    value: this[p],
                    done: this.length === p++
                };
            }
        };
    }
};

// 循环
for (value of myObj) {
    console.log(value);
}
```

输出：
```
a
b
c
```
