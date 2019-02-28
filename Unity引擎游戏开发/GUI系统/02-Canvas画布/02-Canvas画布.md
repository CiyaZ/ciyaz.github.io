# Canvas

Canvas是包含所有的UI控件的父对象，它实际上是一个GameObject，包含了一个叫做`Canvas`的组件。在Unity编辑器里创建一个新的UI控件后，就会自动创建一个Canvas，以后新创建的UI控件都会自动作为Canvas的子对象。Canvas使用EventSystem对象实现消息传递。

## UI控件的绘制顺序

在Hierarchy视图中的顺序就是UI控件的绘制顺序，如果两个UI控件重叠，后面绘制的UI控件就会覆盖前面的。为了动态控制UI控件的覆盖效果，我们可以使用代码来改变UI控件在Hierarchy视图中的顺序，Transform组件的`SetAsFirstSibling`，`SetAsLastSibling`，`SetSiblingIndex`。

## 渲染模式（Render Mode）

Canvas有几种渲染模式可以选择，用来实现不同的效果。

### Screen Space - Overlay

这个模式就是简单的把UI组件绘制到屏幕上，Canvas永远处于绘制的最上层（相对于其他GameObject）。

### Screen Space - Camera

这个模式会把Canvas放到摄像机前，所有UI控件都通过摄像机渲染（也即是说摄像机的参数会影响到Canvas的显示效果，比如摄像机设为透视（Perspective）模式，Canvas的显示就可以做出3D透视效果）。

### World Space

这个模式下Canvas就像是一个Plane，被放置到游戏世界中，GameObject的绘制顺序也是由3D对象的摆放决定的。Canvas的大小可以使用它的`Rect TRansform`手动控制，一些3D游戏里，UI是游戏世界的一部分，此时就要用到这种模式了。
