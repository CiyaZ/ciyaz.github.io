# margin-top 失效问题

关于margin-top，在CSS的盒子模型中有这样的一个规定：如果外层节点和它的第一个内层节点都设置了`margin-top`属性，且外层节点没有padding或border把他们两个分隔开，内层节点的`margin-top`就会将外层节点的该属性覆盖。

毫无道理的一个规定。

下面是例子代码：

```html
<!--设置了margin-top的父div-->
<div style="width: 100%;background: green; margin-top: 100px;">
    <!--设置了margin-top的子div-->
    <div style="width: 80%; height: 50px; background: yellow; margin-top: 200px;"></div>
    <div style="width: 80%; height: 50px; background: red;"></div>
</div>
```

我们想要的到的效果是：外层`div`上边距100px，内层`div`再在内部撑开200px，但实际显示效果却变成了外层`div`上边距为200px，内层并未撑开。

解决办法：

1. 为外层设置`div`设置`overflow: hidden`，推荐使用的解决方案
2. 为外层设置`padding`，但这显然会破坏我们的显示效果，即使只偏移了1像素，而且代码让后人看到也会很费解
3. 父元素或子元素使用浮动定位，这种一般不符合我们布局的需求
