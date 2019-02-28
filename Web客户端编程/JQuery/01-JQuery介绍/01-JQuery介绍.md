# JQuery简介

JQuery是一个javascript函数库，功能包括：

* html元素选择/操作
* dom操作
* css操作
* 事件处理
* 特效和动画
* ajax
* 丰富的插件实现各种特效、功能

# 用途

最初javascript的标准实现的不是很好，浏览器之间互不兼容，接口设计的操作繁琐，因此出现了JQuery等函数库简化javascript开发，而javascript标准实际上也是在不断吸取这些库的经验。至今已经发展到了html5标准的时代，javascript进步了非常多，但是JQuery还是因其易用性保留了下来，至今也是几乎所有网站必备的。

# 使用JQuery

官网：[http://jquery.com/](http://jquery.com/)

下载会得到三个文件：

* jquery-2.1.4.js是未压缩的javascript源码，开发时使用
* jquery-2.1.4.min.js是压缩的javascript代码，项目正式上线时使用，压缩后体积小，加载快，但是无法阅读
* jquery-2.1.4.min.map 源码和压缩代码的关联文件，调试时使用

在页面中引入JQuery：`<script src="js/jquery-2.1.4.min.js"></script>`

从CDN载入：

百度静态资源公共库：[http://cdn.code.baidu.com/](http://cdn.code.baidu.com/)

注意

JQuery是很多其他库的依赖，如Bootstrap，因此一般最先引入。
