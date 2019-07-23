# Image对象

HTML中我们可以使用`<img>`标签加载一个图片，但是对于像游戏、图片处理软件等需求，使用`<img>`加载一个静态的图片是不够的。实际上，浏览器中为JavaScript提供了Image对象，可以实现这个功能。

## 从URL加载图片对象

下面代码使用图片的URL地址初始化了一个Image对象，加载这个图片后，将其绘制在了Canvas中。

```javascript
var canvas = document.getElementById('canvas');
var context = canvas.getContext('2d');

var img = new Image();
img.src = '1.jpg';
img.onload = function () {
    context.drawImage(img, 0, 0);
};
```

## Image对象`<img>`标签

实际上，我们使用`new Image()`创建的对象，可以直接以DOM节点的形式挂载到页面上，页面上的`<img>`标签也可以直接转为`Image`对象。

使用JavaScript加载图片并作为`<img>`标签插入页面：

```javascript
    var div = document.getElementById('div');

    var img = new Image();
    img.src = '1.jpg';
    img.onload = function () {
        div.appendChild(img);
    };
```

页面上会生成这样的对应元素：

```html
<img src="1.jpg">
```

获取页面上的`<img>`标签，并将其绘制在Canvas中：

```javascript
    var img = document.getElementById('img');
    var canvas = document.getElementById('canvas');

    var context = canvas.getContext('2d');
    context.drawImage(img, 0, 0);
```

但是实际上，我们编写普通网页时，创建Image节点还是通常使用`document.createElement('img')`，只有在需要大量图片预加载的Web应用中，才会用到`Image`对象：

```javascript
var img = document.createElement('img');
img.src = '1.jpg';
document.getElementById('div').appendChild(img);
```
