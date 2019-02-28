# 显示/隐藏

* hide(speed, callback)
* show(speed, callback)
* toggle(speed, callback)

参数分别是速度（毫秒），完成时回调函数。

# 淡入/淡出

* fadeIn(speed, callback)
* fadeOut(speed, callback)
* fadeToggle(speed, callback)
* fadeTo(speed, opacity, callback)

其中fadeTo的opacity为淡入淡出至给定透明度，取值0-1之间。

# 滑动

* slideDown(speed, callback)
* slideUp(speed, callback)
* slideToggle(speed, callback)

# 自定义动画

* animate({params},speed,callback);

params定义形成动画的css属性，css属性可以是多个，但必须是camel标记（如css的padding-left要写作paddingLeft），且核心库不提供颜色操作，speed是速度，callback是完成时回调。

```javascript
$("button").click(function(){
  $("div").animate({
    left:'250px',
    opacity:'0.5',
    height:'150px',
    width:'150px'
  });
});
```

# 动画队列

多次调用animate()就会按队列顺序执行，同理链式调用hide()，show()等也能得到相同效果。
