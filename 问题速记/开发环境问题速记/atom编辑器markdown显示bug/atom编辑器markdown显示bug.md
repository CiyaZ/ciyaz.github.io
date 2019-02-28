# atom编辑器markdown显示bug

使用系统默认的markdown语法时，再插入代码块中，遇到C语言写法例如`int *p`会导致编辑界面显示bug，`*`后所有字符被语法分析器解析为斜体，影响了编辑效果。这是`language-gfm`的bug，直到现在也未能解决。唯一的解决办法是，换用`language-markdown`第三方package。

github issue：[https://github.com/atom/language-gfm/issues/44](https://github.com/atom/language-gfm/issues/44)

安装`language-markdown`后，会自动覆盖原本的markdown语法分析器，不会出现上述问题。缺点是markdown的自动提示没有了，不过实际上我们在markdown中也并不需要自动提示。
