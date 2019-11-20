# markdown-md编译器

`markdown`这个库能够将Markdown文本编译成HTML。

## 安装

```
pip3 install markdown
```

## 使用

```python
import markdown

content_rendered = markdown.markdown(content, extensions=['markdown.extensions.extra', 'markdown.extensions.codehilite', 'markdown.extensions.tables'])
```

上面代码中，`content`中存放的是Markdown文本，`extensions`中指定启用了一些插件，像表格支持、语法高亮标识等功能，是需要手动开启的，`content_rendered`是得到的HTML内容。
