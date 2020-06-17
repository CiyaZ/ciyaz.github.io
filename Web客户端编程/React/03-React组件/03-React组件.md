# 定义React组件

这篇笔记我们主要学习以下内容：

* 最简单的静态React组件的写法
* 组件的状态数据使用
* React组件的生命周期

## 编写一个静态的React组件

下面代码中，我们定义一个最简单的静态组件，它仅仅用于展示一段DOM结构，没什么功能。

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

//使用函数定义静态组件，非常简单，推荐使用
function TestComponent() {
    return (
        <h1>Hello, world!</h1>
    );
}
ReactDOM.render(<TestComponent/>, document.getElementById('root'));
```

代码中，我们通过一个函数定义了一个组件，然后使用`ReactDOM.render()`将其渲染到页面上，注意`render()`方法的第一个参数，`<TestComponent />`说明我们引用的是一个组件，而不是一个函数或者其他的什么东西，这是JSX的写法。

除了使用函数定义一个组件，我们也可以通过ES6的类（class）定义一组件。

```javascript
import React, {Component} from 'react';
import ReactDOM from 'react-dom';

//使用ES6的class定义组件类，推荐较复杂的组件使用
class TestComponent extends Component {
    render() {
        return (
            <h1>Hello, world!</h1>
        );
    }
}
ReactDOM.render(<TestComponent/>, document.getElementById('root'));
```

这里要注意，我们使用的是ES6的写法，使用`class`关键字定义组件，除此之外，旧版本的React还可能使用`React.createclass()`，原理是一样的，React推荐使用新的ES6的写法，实际编写时旧版的写法就不要考虑了。

代码中要注意以下几点：

1. 组件必须是大写字母开头
2. `return`返回JSX写法的HTML时，最好用括号括起来，增强代码可读性

## 具有状态数据的组件

下面我们使用React实现一个具有这样功能的组件：按钮点击一次，后面的数字加1。

组件的定义 ClickCount.js
```javascript
import React, {Component} from 'react';

//定义的组件需要继承Component
class ClickCount extends Component {
    constructor() {
        super();
        //state对象用于存储组件的状态（数据）
        this.state = {cnt: 0};
        //将this绑定到btnClick函数。实际上，也可以将btnClick定义为箭头函数，就没有this的问题了。这两种写法都很常见。
        this.btnClick = this.btnClick.bind(this);
    }

    btnClick() {
        //改变组件状态需要使用setState()方法，不要直接操作
        this.setState({cnt: this.state.cnt + 1});
    }

    render() {
        //JSX中，render()的返回值直接写为“(html代码)”的形式
        return (
            <div>
                <button onClick={this.btnClick}>Click</button>
                <span>{this.state.cnt}</span>
            </div>
        );
    }
}

//暴露组件，供外部调用，这个使用的是ES6的组件机制
export default ClickCount;
```

在页面上引入组件的写法 index.js
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import ClickCount from './ClickCount';
import registerServiceWorker from './registerServiceWorker';

ReactDOM.render(<ClickCount />, document.getElementById('root'));
registerServiceWorker();
```

上面代码注释比较详细，这里就不多解释了。

## props和state

一个React组件的数据分为两种，props和state，数据的改变会引发组件的重新渲染。两者的区别是：props是组件对外的数据接口，用于将数据从父组件传递给子组件，state用于维护组件内部的状态数据。

### props 数据的外部接口

下面的例子中，我们通过props将一个字符串数据从父组件传递给子组件，父组件的按钮每点击一次，子组件显示的字符串就多一个字符`A`。

父组件 ParentComponent.js
```javascript
import React, {Component} from 'react';
import ChildComponent from './ChildComponent';

class ParentComponent extends Component {
    constructor() {
        super();
        this.state = {
            childComponentStr: ''
        };

        this.btnClick = this.btnClick.bind(this);
    }

    btnClick() {
        this.setState({childComponentStr: this.state.childComponentStr + 'A'});
    }

    render() {
        return (
            <div>
                <button onClick={this.btnClick}>Click</button>
                <ChildComponent propVal={this.state.childComponentStr}/>
            </div>

        );
    }
}

export default ParentComponent;
```

子组件 ChildComponent.js
```javascript
import React, {Component} from 'react';

class ChildComponent extends Component {
    constructor(props) {
        super(props);
    }

    render() {
        return (
            <div>
                <p>{this.props.propVal}</p>
            </div>
        );
    }
}

export default ChildComponent;
```

props的写法实际上是和`render()`返回的HTML结合起来的，`</>`中组件的属性对应的就是子组件构造函数中的`props`对象的字段。

注意：子组件的编写可能会犯一个错误，子组件这里我们不要多此一举的把`props.propVal`放入`state`中，props对象的更新会触发组件的重新渲染，但是我们在构造函数中把props对象的值赋予state管理，state只初始化了一次，props如何改变组件显示的都是state对象内的值，也就是说页面上显示的值不会改变。

这里可以这样理解：在这里的代码中，实际上子组件的数据状态是由父组件维护的，如果子组件自己再维护一份，那么父组件中数据状态改变就要再通过某种方式通知子组件，那种写法会写很多鸡肋的代码。

### state 数据的内部存储

state理解比较简单，它维护一个组件的状态，说白了就是数据存放在state对象里。

通常我们在组件的构造函数中初始化state对象，组件类中定义的一些方法也会动态的改变state对象中的数据，这些操作都会触发页面上该组件的重新渲染。

最开始的代码中，已经演示过如何动态的修改state对象的内容，我们必须使用`setState()`方法，而不能使用`this.state`直接修改，直接修改会报错，这是React内部机制决定的，`setState()`方法被调用时，除了改变state对象内的数据，也触发了“重新渲染组件”等其他工作。

### state设置存在的坑

注意：`setState()`是个异步操作，如果我们的一个功能需要`setState()`之后紧跟着取出`this.state`，一定要将后一步操作写在`setState()`的回调参数里：

错误写法：
```javascript
handleQuery(params) {
    this.setState({...this.state, fileName: params.fileName, existOption: params.existOption});
    this.reloadTable();
}
```

正确写法：
```javascript
handleQuery(params) {
    this.setState({...this.state, fileName: params.fileName, existOption: params.existOption}, () => {
        this.reloadTable();
    });
}
```

注：`fileName`和`existOption`是两个动态查询参数，`reloadTable()`会立即从`state`取出查询参数执行ajax操作。

## 组件的生命周期

React中，组件的声明周期主要分为三部分：

1. 装载（mount）：组件第一次在DOM树中渲染
2. 更新（update）：组件被重新渲染
3. 卸载（unmount）：组件从DOM树中删除

每个声明周期过程中，都有一些回调函数可供我们使用。其实上面父组件调用子组件方法的示例代码中，我们已经使用了其中的一个回调函数`componentDidMount()`。

<table>
  <tr>
    <th colspan="2">mount(装载)过程</th>
  </tr>
  <tr>
    <td>constructor</td>
    <td>构造函数，通常在这里初始化state对象，以及绑定成员方法的this指向</td>
  </tr>
  <tr>
    <td>getInitialState</td>
    <td>使用ES6语法无效</td>
  </tr>
  <tr>
    <td>getDefaultProps</td>
    <td>使用ES6语法无效</td>
  </tr>
  <tr>
    <td>componentWillMount</td>
    <td>组件渲染前回调</td>
  </tr>
  <tr>
    <td>render</td>
    <td>通常是必须的，通过JSX的HTML代码指定组件如何渲染</td>
  </tr>
  <tr>
    <td>componentDidMount</td>
    <td>组件渲染完成后回调，经常用到</td>
  </tr>
</table>

<table>
  <tr>
    <th colspan="2">update(更新)过程</th>
  </tr>
  <tr>
    <td>componentWillReceiveProps(nextProps)</td>
    <td>组件重新接收props对象时调用</td>
  </tr>
  <tr>
    <td>shouldComponentUpdate(nextProps, nextState)</td>
    <td>这个函数通常用来指定什么时候渲染该组件，什么时候不渲染，返回值为true或false</td>
  </tr>
  <tr>
    <td>componentWillUpdate</td>
    <td>组件重新渲染前回调</td>
  </tr>
  <tr>
    <td>render</td>
    <td>通常是必须的，通过JSX的HTML代码指定组件如何渲染</td>
  </tr>
  <tr>
    <td>componentDidUpdate</td>
    <td>组件重新渲染完成后回调，经常用到</td>
  </tr>
</table>

<table>
  <tr>
    <th colspan="2">unmount(卸载)过程</th>
  </tr>
  <tr>
    <td>componentWillUnmount</td>
    <td>组件卸载前调用</td>
  </tr>
</table>
