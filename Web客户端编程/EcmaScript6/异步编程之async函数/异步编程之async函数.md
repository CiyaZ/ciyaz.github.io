# 异步编程之async函数

ES2017（即ES8）中引入了`async`和`await`两个关键字，专用于编写异步逻辑。异步函数实际上是对`Generator`函数和`Promise`进行封装的一个语法糖，写法可读性非常强。

## async异步函数

在`Generator函数`章节中，我们使用生成器函数，能够将传统的嵌套异步代码转换为顺序代码，但我们依然需要循环调用生成器函数的`next()`，还不是十分方便。使用`async`函数，能够进一步简化代码。

这里，我们将`Generator函数`章节中的代码改为`async`函数写法（略去funcA、funcB、funcC定义部分）：

```javascript
async function asyncFunc() {
    const resultA = await funcA();
    console.log('rsp', resultA);
    const resultB = await funcB();
    console.log('rsp', resultB);
    const resultC = await funcC();
    console.log('rsp', resultC);
}

asyncFunc();
```

这个就非常强了！我们甚至读取返回值的逻辑也不需要放到`then()`中了，异步处理逻辑，全部都是顺序的。

注意：上面我们`async`函数中读取的`resultA`等，都是Promise执行成功的返回值，那么如何在`async`函数中读取Promise执行失败的值呢？这可以使用`try...catch...`来实现。


## async函数返回值

`async`函数返回值也是一个Promise对象，因此我们可以直接使用`then()`等函数进行后续处理，`async`函数中，`return`的内容就是Promise对象`then()`函数的参数。

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
