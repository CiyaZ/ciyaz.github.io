# File API

html5新增了能够读取本地文件的API，同时改进了以前的上传文件表单控件，能够支持多文件上传。

## FileList和file

html5中，对于file控件，`<input type="file" />`，可以指定multiple属性，这样就能上传多个文件。用户选择的每一个文件都是一个file，这些file存储在FileList中。

file属性：

* name 文件名，不包括路径
* lastModifiedDate 文件最后修改时间，Date类型

文件是不能主动读取的，而是只能响应用户的操作，这是问了安全的考虑。

## Blob

file继承自Blob，表示二进制数据。

Blob属性：

* size 字节长度
* type MIME类型，未知类型返回空字符串

## FileReader接口

FileReader可以把文件异步读入内存。

方法：

* readAsText(file, encoding) 文本形式读入文件，编码默认位utf-8
* readAsDataURL(file) dataURL形式读入文件
* readAsArrayBuffer(file) 二进制形式读入文件，返回ArrayBuffer对象
* abort() 中断读取操作

事件：

* onabort 数据读取中断时触发
* onerror 数据读取出错时触发
* onloadstart 数据开始读取时触发
* onprogress 数据读取中周期性触发
* onload 数据读取成功完成
* onloadend 数据读取完成（无论成功失败）

属性：

* error 读取的错误状态，DOMError类型
* readyState 当前读取状态
  * EMPTY 0 未加载数据
  * LOADING 1 数据正在加载
  * DONE 2 已完成全部读取请求
* 读取到的内容，类型取决于调用的读取方法

## 实例代码

### 图片预览
```html
<body>
	<input type="file" id="img_file" />
	<button onclick="preview()">预览</button>
	<img style="width: 100px; height: 100px;" id="img" src=""/>
	<script>
		function preview()
		{
			var img = document.querySelector("#img");
			var file = document.querySelector("#img_file").files[0];
			var reader = new FileReader();

			reader.onload = function()
			{
				img.src = reader.result;
			};

			reader.readAsDataURL(file);
		}
	</script>
</body>
```

这个程序将图片以dataURL形式读入，读入完成后，将其指定给img标签，完成图片的显示。运行结果如下：

![](res/1.png)
