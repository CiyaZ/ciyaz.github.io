# animate.css 动画库

`animate.css`是一个包含很多预设CSS动画效果的库，效果不错，而且使用非常简单。

Github地址：[https://github.com/daneden/animate.css](https://github.com/daneden/animate.css)

我们可以直接在网页上引用，或者通过`npm`等包管理器安装：

```
npm install animate.css --save
```

## 使用

`animate.css`中，各种动画效果都独立定义了`class`，我们在标签上引入即可。

例子：
```html
<p class="animated fadeIn">Hello, world!</p>
```

使用该库的标签必须加上`animated`，然后加上对应效果的标签。上面例子中，我们制造了一个淡入的效果。

## 控制标签

如果需要循环播放，可以加上`infinite`。例如：
```html
<p class="animated fadeIn infinite">Hello, world!</p>
```

如果需要延迟几秒播放，可以使用`delay-1s`至`delay-5s`。例如：
```html
<p class="animated fadeIn delay-5s">Hello, world!</p>
```

## 所有可用效果

| Class Name        |                    |                     |                      |
| ----------------- | ------------------ | ------------------- | -------------------- |
| `bounce`          | `flash`            | `pulse`             | `rubberBand`         |
| `shake`           | `headShake`        | `swing`             | `tada`               |
| `wobble`          | `jello`            | `bounceIn`          | `bounceInDown`       |
| `bounceInLeft`    | `bounceInRight`    | `bounceInUp`        | `bounceOut`          |
| `bounceOutDown`   | `bounceOutLeft`    | `bounceOutRight`    | `bounceOutUp`        |
| `fadeIn`          | `fadeInDown`       | `fadeInDownBig`     | `fadeInLeft`         |
| `fadeInLeftBig`   | `fadeInRight`      | `fadeInRightBig`    | `fadeInUp`           |
| `fadeInUpBig`     | `fadeOut`          | `fadeOutDown`       | `fadeOutDownBig`     |
| `fadeOutLeft`     | `fadeOutLeftBig`   | `fadeOutRight`      | `fadeOutRightBig`    |
| `fadeOutUp`       | `fadeOutUpBig`     | `flipInX`           | `flipInY`            |
| `flipOutX`        | `flipOutY`         | `lightSpeedIn`      | `lightSpeedOut`      |
| `rotateIn`        | `rotateInDownLeft` | `rotateInDownRight` | `rotateInUpLeft`     |
| `rotateInUpRight` | `rotateOut`        | `rotateOutDownLeft` | `rotateOutDownRight` |
| `rotateOutUpLeft` | `rotateOutUpRight` | `hinge`             | `jackInTheBox`       |
| `rollIn`          | `rollOut`          | `zoomIn`            | `zoomInDown`         |
| `zoomInLeft`      | `zoomInRight`      | `zoomInUp`          | `zoomOut`            |
| `zoomOutDown`     | `zoomOutLeft`      | `zoomOutRight`      | `zoomOutUp`          |
| `slideInDown`     | `slideInLeft`      | `slideInRight`      | `slideInUp`          |
| `slideOutDown`    | `slideOutLeft`     | `slideOutRight`     | `slideOutUp`         |
| `heartBeat`       |
