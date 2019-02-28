# Markdown

Markdown是一种简单的标记语言，通常编译为html再由浏览器渲染，十分适合记笔记。不同的Markdown编辑器可能标准不同，但是由于Github中，README的广泛使用，使得Github的Markdown几乎成为事实标准。

本文是Markdown的cheatsheet，列出了Markdown的常用语法。

## 标题

```Markdown
# h1
## h2
### h3
#### h4
##### h5
###### h6
under-line style h1
======
under-line style h2
------
```

## 强调

```Markdown
*斜体* or _斜体_
**粗体** or __粗体__
结合 **粗体 和 _斜体_**
~~删除线~~
```

## 列表

```Markdown
* 无序列表1
* 无序列表2
  1. 无序列表中的有序子列表
* 无序列表3
  * 无需子列表

1. 有序列表1
2. 有序列表2
  * 有序列表中的无序子列表
3. 有序列表3
  1. 有序子列表
```

## 超链接

```Markdown
[百度](http://www.baidu.com)
```

## 图片

```Markdown
![](../res/1.jpg)
```

## 代码

行内代码：`npm install`

代码块：
```python
for i in range(0, 1):
  print("hello")
```

## 表格

```Markdown
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
```

## 引用

```Markdown
> 这是引用
```

## 分割线

```Markdown
分割线
***
```

## 其他规则

* 换行：两次Enter才能换行（也就是两个段落需要间隔一个空行）
* HTML兼容：可以配合使用HTML完成Markdown无法完成的效果（比如插入HTML5视频）
* 链接：允许http的URL，本地文件的URL，相对URL
