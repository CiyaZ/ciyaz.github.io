# Web Storage

用cookie存储一些数据时，cookie比较小，存取可能也不太方便（随http一起发送），因此出现了Web Storage。

WebStorage简单来说，就是操作一些键值对。

* session storage：保存在session中，一次会话的临时保存
* local storage：保存在本地磁盘中，永久保存

## Web Storage存取

* length属性 表示storage大小
* key(i) 根据索引获得键
* getItem(key) 根据键得到值
* setItem(key) 设置键值对
* removeItem(key) 删除键值对
* clear() 清空storage对象
* storage事件 storage改变时触发

## 例子代码

存入session storage的留言本例子
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
	<meta charset="UTF-8">
	<title>Storage Demo</title>
</head>
<body>
	<label for="text">留言</label>
	<textarea id="text"></textarea>
	<button id="btn_submit">确认</button>
	<button id="btn_clear">清空</button>
	<button id="btn_load">加载</button>
	<div id="history"></div>

	<script>
		var text_area = document.querySelector("#text");
		var button_submit = document.querySelector("#btn_submit");
		var button_clear = document.querySelector("#btn_clear");
		var button_load = document.querySelector("#btn_load");
		var history_div = document.querySelector("#history");

		button_submit.addEventListener("click", function(event)
		{
			var data = text_area.value;
			var date = new Date();
			sessionStorage.setItem(date.toLocaleTimeString(), data);
		}, false);

		button_clear.addEventListener("click", function(event)
		{
			sessionStorage.clear();
		}, false);

		button_load.addEventListener("click", function(event)
		{

			history_div.innerHTML = "";

			for(var i = 0; i < sessionStorage.length; i++)
			{
				var key = sessionStorage.key(i);
				var value = sessionStorage.getItem(key);

				var new_div = document.createElement("div");
				new_div.innerHTML = key + " " + value;
				history_div.appendChild(new_div);
			}
		}, false);
	</script>
</body>
</html>
```

## 在firefox中查看Web Storage

在设置中启用存储选项（左下角）

![](res/1.png)

点击存储标签，sessionStorage的内容就在“会话存储”中

![](res/2.png)

## 安全性

WebStorage中和cookie一样，要注意xss的问题，不要将敏感信息存入其中，合理选择session storage还是local storage。
