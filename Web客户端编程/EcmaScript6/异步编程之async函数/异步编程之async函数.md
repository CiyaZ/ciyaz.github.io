# 异步编程之async函数

前面我们介绍了Generator函数配合协程式写法，能够更优雅的实现嵌套回调。ES2017（ES8）中引入了`async`和`await`两个关键字，对Generator函数进行了封装，写法可读性更强。

## 例子

下面两个例子定义的函数作用是相同的：

Generator函数写法：
```javascript
function* operationA() {
    yield operation1();
    yield operation2();
    yield operation3();
}

let opA = operationA();
opA.next();
opA.next();
opA.next();
```

```javascript
async function operationB() {
    await operation1();
    await operation2();
    await operation3();
}

operationB();
```

## async函数返回值的使用

async函数返回值是一个Promise对象，因此我们可以直接使用`then()`等函数进行后续处理，async函数中，return的内容就是Promise对象`then()`函数的参数。

```javascript
async function getTeacherByStudent(student) {
    // 异步调用1
    const classRoom = await getClassRoom(student);
    // 异步调用2
    const teacher = await getTeacherByClassRoom(classRoom);
    return teacher;
}

getTeacherByStudent('Tom').then((resp) => {
    // 一些操作
});
```
