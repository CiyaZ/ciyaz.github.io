# 异步编程之Promise

Promise是一种异步编程模型，主要用来解决大量回调函数嵌套问题，Promise并不能让代码更加简洁，相反它会让代码变得更复杂，但是复杂并不意味着困难，相反使用Promise取代大量回调函数嵌套能够让代码编写变得简单。

大量回调函数嵌套可读性非常差，使用Promise能提高代码可读性，帮助开发者理清思路。但是前提是代码的读者也学习过Promise模式，并十分熟悉异步编程，否则可读性就无从谈起了。

下面是一个简单的Promise使用例子，代码通过Promise实现了回调。

```javascript
let promise = new Promise((resolve, reject) => {
    setTimeout(()=>{
        resolve('执行成功');
    }, 1000);
});

promise.then((success)=>{
        console.log(success);
    });
```

输出：
```
执行成功
```

## Promise对象的三种状态

* `pending`：未确定状态，异步操作的执行的逻辑还没有一个确定的结果
* `fulfilled`；异步操作成功
* `rejected`：异步操作失败

在这三种状态中，只能从`pending`转化为`fulfilled`，或`pending`转化为`rejected`，这是显而易见的。

## Promise.prototype.then()

`then(onfulfilled, onrejected)`函数用于为Promise模型的异步操作绑定回调函数，它接收两个函数作为参数：

* `onfulfilled`：Promise对象中的异步操作执行成功时回调（`resolve()`被调用）
* `onrejected`：Promise对象中的异步操作执行失败时回调（`reject()`被调用，一般不在`then()`函数中使用这个参数，而是在`catch()`中）

## Promise.prototype.catch()

`catch(onrejected)`和`then()`函数其实是一样的，只不过没有第一个参数`onfulfilled`，仅能用于捕获失败的情况。

上面介绍过then函数有两个参数，我们可以向then函数传入两个回调函数参数，分别用来处理异步操作执行成功和执行失败的情况，但是这样写不太美观，推荐使用`catch()`函数代替`then()`的第二个参数。如果`then()`函数没有指定执行失败的回调，它就会把Promise对象一步步传下去，直到链式调用遇到`catch()`函数。

## `then()`函数的返回值

`then()`的返回值是一个`Promise`对象，根据回调函数`onfulfilled`的返回值，它稍微有点复杂：

1. 如果回调函数不返回任何值，`then()`返回一个`fulfilled`的`Promise`，其值为`undefined`
2. 如果回调函数返回一个值，`then()`返回一个`fulfilled`的`Promise`，其值为回调函数的返回值
3. 如果回调函数抛出错误，`then()`返回一个`rejected`的`Promise`，值为回调函数抛出的值
4. 如果回调函数返回一个`Promise`，`then()`也返回一个相同状态的`Promise`

因为返回的也是`Promise`，因此我们能够在`then()`上实现链式调用，`onfulfilled`函数的返回值会决定链式调用的下一步`Promise`状态。

这个不太好描述，我们直接写一个例子：

```javascript
let promise = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('success message 1');
    }, 1000);
});

promise
    //返回一个字符串，Promise变为fulfilled状态
    .then((success) => {
        console.log(success);
        return 'success message 2';
    })
    //抛出一个异常，Promise变为rejected状态
    .then((success) => {
        console.log(success);
        throw 'error message 3';
    })
    //返回一个新Promise对象，该Promise对象执行成功，变为fulfilled状态
    .catch((err) => {
        console.log(err);
        return new Promise((resolve, reject) => {
            resolve('success message 4');
        })
    })
    //返回一个新Promise对象，该Promise对象执行失败，变为rejected状态
    .then((success) => {
        console.log(success);
        return new Promise((resolve, reject) => {
            reject('error message 5');
        })
    })
    //什么都不返回，Promise还是fulfilled状态
    .catch((err) => {
        console.log(err);
    })
    //即使上一步什么都不返回，then也能成功接收，但是传入的参数是undefined
    .then((success) => {
        console.log(success);
    });
```

我们可以想象一下，如果使用嵌套回调的写法，上面代码要嵌套很多很多层，而改为Promise写法，嵌套一层转化为了多写一个链式调用，因此代码可读性就变好了。

## Promise.all()

`Promise.all()`函数用于执行多个异步操作，当所有异步操作全部执行成功时，Promise对象变为`fulfilled`，否则变为`rejected`。

```javascript
let arr = ['aaa', 'bbb', 'ccc'];
let promiseArr = arr.map((val) => {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(val);
        }, 1000);
    });
});

Promise.all(promiseArr).then((success) => {
    console.log(success);
});
```

结果用时1s（因为是全部是异步的），输出的结果`success`实际上是一个数组：
```
[ 'aaa', 'bbb', 'ccc' ]
```

## Promise.race()

`Promise.race()`和`all()`差不多，`race()`也接收多个异步操作，但区别是如果`race()`中某个异步操作率先完成，Promise对象会变为该异步操作的`fulfilled`或`rejected`状态。
