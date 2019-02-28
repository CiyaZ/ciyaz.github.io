# JQuery插件开发

JQuery实际上就是一个函数库，JQuery插件开发实际非常简单，就是给JQuery对象加函数。

首先我们要知道JQuery中，方法都在`jQuery.fn`上定义，而且`jQuery.fn=jQuery.prototype`，因此JQuery对象可以通过原型继承这些方法。我们开发JQuery插件，只要在`jQuery.fn`的基础上进行扩展就行了。

## 一个简单的例子

下面例子，以JQuery插件的形式，写了一个改变标签CSS的函数。

test_plugin.js
```javascript
(function($){
	$.fn.changeStyle = function(colorStr){
		this.css("color",colorStr);
		return this;
	}
}(jQuery));
```

我们的代码中，仅仅是在`$.fn`上定义了一个函数。这里的this，指向的是当前调用插件函数的JQuery对象。

注：为了满足JQuery链式调用的特色，函数最后返回了这个JQuery对象（this）。

调用JQuery插件
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>JQuery Plugin Demo</title>
	<script src="jquery-3.2.1.min.js"></script>
	<script src="test_plugin.js"></script>
</head>
<body>
<p>aaa</p>
</body>
<script>
	$("p").changeStyle("red");
</script>
</html>
```

## 下雪插件例子

下面是个稍微复杂的例子，实现网页上飘雪花的效果（代码是网上找到的）。

jq_snow.js
```javascript
(function ($) {

	$.fn.snow = function (options) {

		var $flake = $('<div id="snowbox" />').css({'position': 'absolute', 'top': '-50px'}).html('&#10052;'),
			documentHeight = $(document).height(),
			documentWidth = $(document).width(),
			defaults = {
				minSize: 10,		//雪花的最小尺寸
				maxSize: 20,		//雪花的最大尺寸
				newOn: 1000,		//雪花出现的频率
				flakeColor: "#FFFFFF"
			},
			options = $.extend({}, defaults, options);

		var interval = setInterval(function () {
			var startPositionLeft = Math.random() * documentWidth - 100,
				startOpacity = 0.5 + Math.random(),
				sizeFlake = options.minSize + Math.random() * options.maxSize,
				endPositionTop = documentHeight - 40,
				endPositionLeft = startPositionLeft - 100 + Math.random() * 500,
				durationFall = documentHeight * 10 + Math.random() * 5000;
			$flake.clone().appendTo('body').css({
				left: startPositionLeft,
				opacity: startOpacity,
				'font-size': sizeFlake,
				color: options.flakeColor
			}).animate({
					top: endPositionTop,
					left: endPositionLeft,
					opacity: 0.2
				}, durationFall, 'linear', function () {
					$(this).remove()
				}
			);

		}, options.newOn);

	};

})(jQuery);
```

调用
```javascript
$(function(){
  $.fn.snow({
    minSize: 5,
    maxSize: 50,
    newOn: 300
  });
});
```

实际上代码很简单，基本原理就是随机将雪花div画到屏幕上，使用`setInterval()`循环更新雪花的位置，并利用JQuery的动画效果，实现淡出。这里传了一个`options`对象作为参数，而且函数内部还定义了一个`defaults`对象，并使用`$.extend()`将二者结合。

注：其实有个bug，这个插件雪花横着飘会影响浏览器的横向滚动条。
