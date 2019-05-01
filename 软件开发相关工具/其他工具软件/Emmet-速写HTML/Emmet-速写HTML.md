# Emmet 速写HTML

HTML语法和XML比较像，也同样的繁琐，各种尖括号标签、属性等，如果纯手写会让人不厌其烦。Emmet（曾经叫ZenCoding）用于快速生成一段HTML文本，最初是一个广泛使用的插件，由于它实在太流行了，后来Webstorm、VSCode等主流开发工具都默认内置支持Emmet（据说Dreamweaver都支持）。

Emmet能够一定程度上加快码页面的速度，但是当下的各种开发工具也都有良好的代码提示支持，一些Emmet的高级用法很是鸡肋，这篇笔记记录最常用的用法。

## 一个例子

在VSCode中编写如下内容，然后按下`Tab`，Emmet就会生效，自动生成HTML文本。
```
div.container>div.panel>ul#nav>li*5
```

```html
<div class="container">
    <div class="panel">
        <ul id="nav">
            <li></li>
            <li></li>
            <li></li>
            <li></li>
            <li></li>
        </ul>
    </div>
</div>
```

## 标签id和class

类似CSS选择器，使用`标签.类名`的形式，生成一个带有指定class属性的标签。

```
div.container
```

```html
<div class="container"></div>
```

使用`标签#类名`的形式，生成一个带有指定id属性的标签。

```
div#app
```

```html
<div id="app"></div>
```

## 子标签节点

使用`>`指定一个标签节点的子节点。

```
div>span
```

```html
<div><span></span></div>
```

## 多个子节点

多个同样的子节点使用`*n`的形式自动生成。

```
div>span*5
```

```html
<div>
    <span></span>
    <span></span>
    <span></span>
    <span></span>
    <span></span>
</div>
```

## 兄弟节点

多个标签节点使用`+`连接。

```
div>span.col1+span.col2+span.col3
```

```html
<div>
    <span class="col1"></span>
    <span class="col2"></span>
    <span class="col3"></span>
</div>
```
