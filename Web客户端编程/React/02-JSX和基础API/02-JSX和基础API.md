# JSX和基础API

这篇笔记作为React的入门，我们学习一些React中最简单的概念。

## JSX文件

在React中，我们使用React专门定义的JSX写法定义组件。JSX是JavaScript的语法扩展，它的语法专门适用于编写React组件，我们可以使用`.js`或是`.jsx`作为文件后缀，大部分IDE或是文本编辑器都能正确识别。

jsx和js文件的主要区别就是jsx中我们能直接混合HTML标签编写代码，比如像下面这样：

```javascript
let a = <div>hello, world!</div>;
ReactDOM.render(a, document.getElementById('root'));
```

甚至CSS也是可以混合使用的：
```javascript
let aStyle = {color: 'red'};
let a = <div style={aStyle}>hello, world!</div>;
```

我们混合写了html，js，css，这和原本三者文件分离的原则相违背，但是React实际上并不是开历史的倒车，React是组件化的，每个组件都有自己的html，js，css，使用JSX把他们写到一起正是高内聚低耦合的体现。不管怎么说，仅从实际编程体验来讲，使用组件化思想结合JSX的写法一般都是好处大于坏处的。

实际上我们并不需要特别再学什么内容，JSX的写法照葫芦画瓢就行了，我们使用React开发应用，原因一定是：需求要求我们把关注点放在如何堆砌组件展示、处理数据，这也是为什么我们选择组件化的前端框架而不是JQuery。

## 将元素渲染到DOM

上面例子代码中已经给出如何将静态数据渲染到DOM中了：

```javascript
let a = <div>hello, world!</div>;
ReactDOM.render(a, document.getElementById('root'));
```

`ReactDOM.render(element, container)`一般接收两个参数，第一个参数是要渲染的DOM结构，第二个参数是在哪里渲染。

## 在DOM结构上更新元素

参考官网例子，下面代码我们实现了一个计时器：

```javascript
function tick() {
    const element = (
        <div>
            <h1>It is {new Date().toLocaleTimeString()}.</h1>
        </div>
    );
    ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
```

`setInterval()`函数定时调用`tick()`函数，每次`tick`被调用都会触发`ReactDOM.render()`使得组件被重新渲染。实际上，React只会重新渲染DOM结构改变了的部分，而不会刷新整个DOM树，React内部实现了一个Virtual DOM，它会使用一个差异算法计算哪些DOM子树需要更新，因此具有较好的渲染性能。

上面代码仅仅是说明如何将DOM元素渲染到页面上，因为这种做法并不符合组件化的思想，这样写就和JQuery没什么区别了。下一篇笔记中，我们介绍如何定义React组件。
