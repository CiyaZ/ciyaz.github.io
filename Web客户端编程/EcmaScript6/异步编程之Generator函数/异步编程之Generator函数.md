# 异步编程之Generator函数

ES6中引入了Generator（生成器）函数。JavaScript语言中存在大量的嵌套回调写法，如果使用Generator配合Promise的写法，能帮助我们理清异步操作的逻辑，极大的提升我们代码的可读性，这篇笔记我们主要记录JavaScript中的Generator函数用法。

## Generator函数

什么是Generator函数？我们直接看一个例子：

```javascript
function *foo() {
    yield 1;
    yield 2;
    return 3;
}

let fn = foo();
console.log(fn.next());
console.log(fn.next());
console.log(fn.next());
console.log(fn.next());
```

Generator函数需要使用`*`定义，其内部使用`yield`关键字输出函数运行过程中的一个状态，在外部（调用函数的地方）使用`next()`函数进行访问，直到函数`return`。

输出
```
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: true }
{ value: undefined, done: true }
```

Generator函数的返回值是一个对象，`value`是`yield`返回的值，`done`表示该函数是否`return`，函数已经返回后，`value`将变为`undefined`。

### for...of... 迭代Generator函数输出

我们可以使用ES6引入的`for...of...`迭代Generator函数的输出：

```javascript
function *foo() {
    yield 1;
    yield 2;
    return 3;
}

let fn = foo();
for(let f of fn) {
    console.log(f);
}
```

输出
```
1
2
```

### Generator.prototype.next()

`next()`函数可以传一个参数，用于覆盖上一次`yield`返回的值：

```javascript
function *loop() {
    let ctl = true;
    while (ctl) {
        ctl = yield ctl;
    }
}

let fn = loop();
console.log(fn.next(false));
console.log(fn.next());
```

注意上面代码的`ctl = yield ctl`，我们不仅使用yield返回了一个函数的内部状态参数，还进行了一次赋值。执行过程是这样的：

1. 第一次调用`next(false)`，ctl初始值为true，执行循环，当执行到`yield ctl`时，返回`{ value: true, done: false }`
2. `ctl`被赋值为`next(false)`的参数`false`
3. `while(ctl)`开始第二轮循环，`ctl`为`false`，循环结束，函数结束（无return）
4. 第二次调用`next()`，函数已经执行结束了，返回`{ value: undefined, done: true }`

由此可见，我们通过`next()`不仅可以读取函数的内部状态，还能改变函数的内部状态。

### Generator.prototype.throw()

`throw()`函数能向Generator函数内部在yield位置抛一个异常，函数可以可以用`try...catch...`接收。

```javascript
function* foo() {
    try {
        yield;
    } catch (e) {
        console.log(e);
    }
}

let fn = foo();
console.log(fn.next());
console.log(fn.throw('run time error'));
console.log(fn.next());
```

上面代码中，我们首先调用`next()`，让函数执行到`yield`位置，然后调用`throw`，向其扔一个异常，函数的后续执行流程就会被改变，进行错误处理相关的代码。

### Generator.prototype.return()

`return()`能够返回给定的值，并立即终结函数的运行。

例子：
```javascript
function* foo() {
    while(true) {
        yield;
    }
}

let fn = foo();
console.log(fn.next());
console.log(fn.return('aaa'));
console.log(fn.next());
console.log(fn.next());
```

输出：
```
{ value: undefined, done: false }
{ value: 'aaa', done: true }
{ value: undefined, done: true }
{ value: undefined, done: true }
```

### Generator函数嵌套

Generator函数嵌套可以使用`yield*`表示。

```javascript
function* foo() {
    yield 1;
    yield* bar();
}

function* bar() {
    yield 2;
}

let fn = foo();
console.log(fn.next());
console.log(fn.next());
console.log(fn.next());
```

注意：`*`符号的位置不要和C++搞混：

* C++中`int *p`表示`p`是一个指针类型，指向整数，而没有整数指针类型这种说法，因此建议写成`int *p`而不是`int* p`
* JavaScript中，`function*`表示表示定义Generator函数，`yield*`也是差不多意思

说实话，C++的说法确实比较绕，但是毕竟是老牌编程语言，基本是个程序员都会，可能一些概念先入为主，这里注意不要把自己搞晕了。

## Generator配合Promise异步编程

使用Generator函数单独的编写同步式代码，可能没什么特别的优势，但编写异步代码时，其作用就凸显出来了。

假设我们有三个异步请求`funcA`、`funcB`、`funcC`，需求要求先调`funcA`，再调`funcB`，最后调`funcC`。如果使用传统的写法，我们不得不嵌套着写三个函数的逻辑，如果只使用Promise，也势必会写一串`then()`的链式调用，那么使用Generator函数此时如何实现呢？

```javascript
function funcA() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve('funcA return');
        }, 1000);
    });
}

function funcB() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve('funcB return');
        }, 1000);
    });
}

function funcC() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve('funcC return');
        }, 1000);
    });
}

function* gen() {
    // 顺序编写异步逻辑
    yield funcA();
    yield funcB();
    yield funcC();
}

const g = gen();
function next() {
    const result = g.next();
    if (result.done) {
        return;
    }
    result.value.then((rsp) => {
        console.log('rsp', rsp);
        next();
    });
}

next();
```

如上面代码所示，我们在生成器函数`gen()`中顺序的定义了三个异步操作，它们的返回值都是`Promise`，我们循环调用生成器函数时，只要在`then()`中调用生成器函数的`next()`，就能实现异步逻辑的顺序编写，异步执行了。这样，我们的代码也变得简单易懂，可读性更强了。
