# 使用Vuex

Vuex用来在Vue中实现单向数据流，适用于状态管理极多，页面数据结构非常复杂的大型前端应用（其实一般XX管理系统都属于这个范畴）。React中我们经常使用Redux实现类似功能，Vuex和Redux略有不同，相比Redux进行了一些改进，使用更加简单优雅，而且容易理解，如果会Redux就能立刻上手使用Vuex。

这里我们参考官方文档学习一下Vuex。

[https://vuex.vuejs.org/zh/](https://vuex.vuejs.org/zh/)

## 安装

`vue-cli3`下，我们直接用命令安装`vuex`。

```
vue add vuex
```

安装完成后，`vue-cli`会自动为我们创建一个`store.js`，并在入口文件中注册到Vue实例，我们可以在其基础上测试。

## Store和State

Vue是数据驱动页面的，Vuex实现了单向数据流，组件数据统一放在Vuex的Store中保存，存储数据的对象我们叫State。下面是一个计数器的例子，每点一次按钮计数值加1。

store.js
```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    cnt: 0
  },
  mutations: {
    inc: function(state) {
      state.cnt++;
    }
  }
})
```

上面代码中就是实例化的一个Store，其中`state.cnt`就是我们保存的数据。Vuex中，通过提交`mutation`的方式对数据进行同步修改。

Cnt.vue
```html
<template>
    <div>
        <div v-text="cnt"></div>
        <button @click="inc">+</button>
    </div>
</template>
<script>
export default {
    name: 'Cnt',
    computed: {
        cnt: function() {
            return this.$store.state.cnt;
        }
    },
    methods: {
        inc: function() {
            this.$store.commit('inc');
        }
    }
}
</script>
```

注意读取和设置Store中数据的方法，我们通过`this.$store`访问Store对象，读取直接读就行了，我们读取Store数据并放在计算属性中，就可以在组件模板中访问了。

而设置State必须通过`commit()`提交`mutation`实现，参数就是`mutation`的函数名。

## Getter

我们直接读`this.$store.state`，这个数据可能还有一些后续处理步骤，这些逻辑如果数据数据模型本身而不是渲染逻辑，都写在组件的`computed`属性中就不太好了。Store中也支持计算属性，只不过它叫`Getter`。

```javascript
export default new Vuex.Store({
  state: {
    cnt: 0
  },
  getters: {
    cntFormatted: function (state) {
      return '当前值为：' + state.cnt + "GB";
    }
  },
  mutations: {
    inc: function (state) {
      state.cnt++;
    }
  }
})
```

上面例子代码中，我们在`getters`中编写了一些类似组件「计算属性」的逻辑。

```javascript
computed: {
        cntFormatted: function() {
            return this.$store.getters.cntFormatted;
        }
    }
```

组件中，通过上述写法访问Store的Getter。

## Mutation

之前我们简单使用了Mutation提交数据，这里再深入学习一下。

### 提交Mutation

定义Mutation：

```javascript
mutations: {
  inc: function (state) {
    state.cnt++;
  }
}
```

在组件中提交Mutation：

```javascript
this.$store.commit('inc');
```

### 携带载荷

提交Mutation时可以携带一些参数，这些参数通常封装在一个对象中，我们称为`payload`。假设现在我们实现填写一个客户信息表单，点击确认按钮将客户数据提交到Store中，`commit`这个动作就需要携带数据了。

store.js中`Mutation`定义：
```javascript
mutations: {
  addCust: function (state, payload) {
    state.custs.push(payload);
  }
}
```

提交写法：
```javascript
methods: {
  handleClick: function() {
    let payload = {
      custId: this.custId,
      custName: this.custName,
      sex: this.sex,
      age: this.age
    };
    this.$store.commit("addCust", payload);
  }
}
```

其实，我这里将数据用`v-model`绑定到了组件的`data`上，然后组装为`payload`，通过`commit`提交`Mutation`。你也可以在`handleClick`中，读取一次表单值。

## Action

`Mutation`只能进行同步处理，而不能包含异步处理，这个很好理解。如果希望使用异步处理，自然也就需要在`Mutation`上封装一层。带有异步操作的数据更新需要使用`Action`，而`Action`必须通过提交`Mutation`改变数据。

在上面的基础上，假设我们要在组件初始化时，从后台加载已有的客户数据到Store中，就需要在组件的`created()`回调函数里提交一个`Action`。先看`Store`中`Action`的定义：

```javascript
actions: {
  loadTable: function ({ commit }, queryParam) {
    window.console.log(queryParam);
    // 模拟一个异步请求
    setTimeout(function () {
      let cust1 = {
        custId: 1,
        custName: 'tom',
        age: 3,
        sex: 'male'
      };
      let cust2 = {
        custId: 2,
        custName: 'jerry',
        age: 1,
        sex: 'female'
      };
      let cust3 = {
        custId: 3,
        custName: 'lucy',
        age: 9,
        sex: 'female'
      };
      commit("addCust", cust1);
      commit("addCust", cust2);
      commit("addCust", cust3);
    }, 3000);
  }
}
```

这个`Action`其实就是模拟几个客户数据，然后用`setTimeout()`模拟一个异步操作，假装是从后端载入的。数据加载后，我们通过`commit()`提交`Mutation`更改`Store`中数据。

每个Action中，第一个参数是`context`，我们一般通过解构赋值读取其中的几个参数：

```
{
  state,      // 当前模块的State（见后文State多模块）
  rootState,  // 根State
  commit,     // 用于提交Mutation
  dispatch,   // 用于提交其它Action
  getters,    // 访问Store的计算属性，相当于当前模块的store.getters
  rootGetters // 访问Store的计算属性，相当于根模块的store.getters
}
```

第二个参数是`dispatch()`的`payload`。

提交`Action`：
```javascript
this.$store.dispatch("loadTable", "123")
```

其中，`loadTable`是`Action`的名字，`123`是`payload`。

## Module

Vuex将整个项目的数据结构提取出来集中存储，但是我们的项目如果过于复杂，把所有数据结构维护在一起肯定是不行的，还得分模块把它们拆开。

假设我们把`Store`拆为子模块`moduleA`，`moduleB`，再加上根模块本身，一共三部分。子模块的写法和根模块一样，无非就是包含`state`，`mutations`，`actions`，`getters`的一个对象。

```javascript
const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})
```

上面代码实现在根模块中注册子模块。

## Vuex中和Store相关的项目目录结构

安装Vuex时，`vue-cli`自动为我们创建了一个`store.js`但是随着我们项目变得庞大，只用一个`store.js`是远远不够的，我们需要创建一个单独的`store`文件夹，存放相关的内容。例如：

```
store
|_index.js 初始化Store
|_actions.js 定义所有的根Action
|_mutations.js 定义所有的根Mutation
|_module1 模块1相关的文件
  |_index.js 模块1中Store的初始化
  |_actions.js 模块1中所有的Action
  |_mutations.js 模块1中所有的Mutation
|_...
```
