# Less简介和环境搭建

Less是一个CSS扩展语言，我们知道CSS其实就是一种配置文件，它的语法非常简单，但这也给我们造成了一定的麻烦，比如我们要统一为某类组件指定其宽度`width`，CSS中我们不得不在一堆`class`中指定相同的`width`数值，如果我们突然想起来要改一下这个值，就非常困难了。Less中就可以定义变量统一供其他位置引用，解决了这个问题。

Less还有很多方便的特性，能让较复杂的CSS工程编写变得简单，本系列笔记主要介绍一下Less中常用的语法。

## 环境搭建

### less.min.js

最简单的办法是在网页中加载一个即时编译器，将`less`编译成`css`样式并应用到页面上。我们可以直接引入下面代码，也可以下载下来后从本地引入。

```html
<script src="//cdnjs.cloudflare.com/ajax/libs/less.js/3.9.0/less.min.js" ></script>
```

在页面中引入`less`：
```html
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Less Demo</title>
    <link rel="stylesheet" type="text/less" href="style.less" />
    <script src="less.min.js"></script>
</head>

<body>
    <div class="app">hello</div>
</body>

</html>
```

注意：`less.min.js`必须放在所有引入`less`的标签后，这样`less`才能生效，`less.min.js`会在页面上插入一个`style`节点，包含所有`less`文件的编译输出。

### lessc

`less.min.js`在真实环境中比较少用，引入无关紧要的`less`文件和编译器增大了网络传输开销，我们也可以用Less编译器手动编译，并载入页面。

全局安装less编译器：
```
yarn global add less
```

编译一个less文件：
```
lessc style.less style.css
```
