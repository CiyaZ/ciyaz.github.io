# react-redux的使用

这篇笔记介绍如何在React项目中使用`react-redux`整合`redux`库，介绍`react-redux`库中的一些关键API。

## 在项目中安装Redux相关的库

在React项目中使用Redux库，我们至少需要`redux`和`react-redux`两个包。在项目中引入依赖：

```
npm install --save redux react-redux
```

## 使用react-redux实现Counter

相比于前一篇笔记中直接使用Redux的方式，`react-redux`主要为我们实现了：使用`connect()`函数自动生成外层组件（上一篇笔记中的App组件），我们只需要使用`props`接收数据和回调函数即可。

```
connect(mapStateToProps, mapDispatchToProps)(Component)
```

* 参数 mapStateToProps：向内层组件props传入的state数据
* 参数 mapDispatchToProps：向内层组件props传入的回调函数
* 参数 Component：内层组件
* 返回值：外层组件

## 代码实现

这里我们使用`react-redux`改写上一篇笔记中编写的Counter组件。

src/redux/action.js
```javascript
export const INCREMENT = "counter_INCREMENT";
export const DECREMENT = "counter_DECREMENT";
export const RESET = "counter_RESET";

export function increment() {
    return {type: INCREMENT}
}

export function decrement() {
    return {type: DECREMENT}
}

export function reset() {
    return {type: RESET}
}
```

src/redux/reducer.js
```javascript
import {INCREMENT, DECREMENT, RESET} from "./action";

function reducer(state = 0, action) {
    switch (action.type) {
        case INCREMENT:
            return state + 1;
        case DECREMENT:
            return state - 1;
        case RESET:
            return 0;
        default:
            return state;
    }
}

export default reducer;
```

src/redux/store.js
```javascript
import {createStore} from 'redux';
import reducer from './reducer';

let store = createStore(reducer);

export default store;
```

src/redux/container.js
```javascript
import {connect} from 'react-redux';
import Counter from '../component/Counter';
import {increment, decrement, reset} from './action';

const mapStateToProps = (state) => ({
    value: state
});

const mapDispatchToProps = (dispatch) => ({
    handleIncBtnClick: () => dispatch(increment()),
    handleDecBtnClick: () => dispatch(decrement()),
    handleRstBtnClick: () => dispatch(reset())
});

let App = connect(mapStateToProps, mapDispatchToProps)(Counter);

export default App;
```

src/component/Counter.js
```javascript
import React, {Component} from 'react';

class Counter extends Component {
    constructor(props) {
        super(props);
    }

    render() {
        return (
            <div>
                <div>{this.props.value}</div>
                <div>
                    <button onClick={this.props.handleIncBtnClick}>+</button>
                    <button onClick={this.props.handleDecBtnClick}>-</button>
                    <button onClick={this.props.handleRstBtnClick}>C</button>
                </div>
            </div>
        );
    }
}

export default Counter;
```

index.js
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import {Provider} from 'react-redux';
import App from './redux/container';
import store from './redux/store';
import registerServiceWorker from './registerServiceWorker';

ReactDOM.render(
    (<Provider store={store}>
        <App/>
    </Provider>),
    document.getElementById('root'));

registerServiceWorker();
```

## 具有多个reducer函数时

通常项目中，我们不会只有一个reducer函数，而是按照功能把reducer划分成不同的函数，然后使用`combineReducers`将所有的reducer组成起来，用组合后的reducer初始化store。此时，`react-redux`中的处理方式是每一个reducer对应一段store中的数据，reducer函数名和store对象中的属性名相同。

例如：reducer函数叫做`customerAddReducer(state, action)`，那么store中对应的数据（传入reducer的参数）就是`state.customerAddReducer`，每一个reducer都是这样处理的。
