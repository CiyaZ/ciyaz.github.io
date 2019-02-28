# Vue中使用事件

有关JavaScript中的事件，请参考`JavaScript客户端编程/事件处理详解`。在Vue框架中，使用事件和直接用JavaScript稍有不同。

## 获取event对象

```html
<body>
<div id="app">
	<input type="text" @keydown="func">
</div>
<script>
	let app = new Vue({
		el: "#app",
		data: {},
		methods: {
			func: function (event)
			{
				alert(event.keyCode);
			}
		}
	});
</script>
</body>
```

直接在绑定的函数中，加上一个`event`参数，就可以得到event对象了，上面代码中，我们绑定了按键按下事件，从event对象得到了按下的键码。

## 传递参数

```html
<body>
<div id="app">
	<input type="text" @click="func(2)">
</div>
<script>
	let app = new Vue({
		el: "#app",
		data: {},
		methods: {
			func: function (i)
			{
				alert(i);
			}
		}
	});
</script>
</body>
```

传递的参数直接在`@click`绑定时写进去就行了。

## 阻止事件冒泡

我们知道HTML组件之间具有事件冒泡的特性，不使用Vue时，阻止事件冒泡需要在代码设置：

```javascript
event.cancelBubble=true;
```

Vue中阻止事件冒泡使用可以用`.stop`修饰`v-on`指令，写法如下：

```javascript
@click.stop="func"
```

## 阻止默认行为

例如我们自定义右键菜单时，需要阻止默认的浏览器右键菜单，JavaScript代码中会如下设置：

```javascript
event.preventDefault();
```

Vue中，阻止右键点击默认行为的写法更加简单：

```javascript
@contextmenu.prevent="func"
```
