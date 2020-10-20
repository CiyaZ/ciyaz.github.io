# fixed实现水平居中

页面如果需要实现类似Android的`Toast`这种消息提示，这就要求使用`position:fixed`。

![](res/1.png)

我们知道这种定位方式是通过`left`、`top`等属性指定的，但定位的坐标点是`<div>`左上角，弹出内容本身的长度也不固定，直接来个`left: 50%`并不是我们想要的效果。那么如何实现水平居中呢？

## CSS实现

一种方式是使用JavaScript来计算，但显然比较麻烦。最好的方式是使用CSS3的`transform`来实现。

```css
.toast {
    position: fixed;
    left: 50%;
    transform: translateX(-50%);
    display: inline-block;
}
```

上面代码中，我们的弹出信息左边缘在水平位置上偏移`50%`，在此基础上又减去了自身宽度的`50%`，恰好就是水平居中了。

## 一种错误的方式

以前写过一个典型的错误实现：用一个`width:100%`的透明空`<div>`作为`position:fixed`的区块，然后在其内部实现水平居中。

虽然乍一看能实现效果，但实际上空`<div>`把后面的元素都给遮挡了，这样显然是不行的。
