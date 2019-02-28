# Python的json解析库

使用过JavaScript我们知道，JavaScript处理json实在太方便了，因为json字符串和JavaScript对象几乎是一一对应的，只需要一个函数就可以互相转换。Python则有字典和列表结构，其实也是能和json对象一一对应的，Python内置了json的解析库，使用起来和JavaScript一样方便。

下面介绍json模块。

## 解析json字符串

用做例子的json字符串：
```json
{
  "users": [
	{
	  "name": "Tom",
	  "age": "11"
	},
	{
	  "name": "Jerry",
	  "age": "12"
	},
	{
	  "name": "Cate",
	  "age": "10"
	}
  ]
}
```

解析json字符串的代码：
```python
import json

fp = open("test.json", "r")
obj = json.load(fp)
print(obj)
fp.close()
```

运行结果：
```
{'users': [{'name': 'Tom', 'age': '11'}, {'name': 'Jerry', 'age': '12'}, {'name': 'Cate', 'age': '10'}]}
```

真的非常简单，`json.load()`函数传入一个文件描述符，即可把json字符串解析为“字典-列表”结构。

* `json.load(fp)` 传入一个文件描述符，解析文件中的json字符串
* `json.loads(s)` 传入一个json字符串并解析

这两个函数中还有许多实用的关键字参数，这里就不一一解释了。具体请参考Python3的官方文档。

## 序列化json对象

Python中我们需要准备一个“字典-列表”结构，json模块能够识别这种结构并把它序列化为json。

```python
import json

obj = {
	"users": [
		{
			"name": "Tom",
			"age": "11"
		},
		{
			"name": "Jerry",
			"age": "12"
		},
		{
			"name": "Cate",
			"age": "10"
		}
	]
}
json_str = json.dumps(obj)

print(type(json_str))
print(json_str)
```

输出结果：
```
<class 'str'>
{"users": [{"name": "Tom", "age": "11"}, {"name": "Jerry", "age": "12"}, {"name": "Cate", "age": "10"}]}
```

等等，obj不是把上面例子1文件里的json复制了一遍吧？就是这样，因为json对象大括号就对应Python字典大括号，json数组就对应Python列表。为了证明输出结果是字符串，我使用了`type()`判断了输出的对象类型。

* `json.dump(obj, fp)` 第一个参数是json字典，第二个参数是文件描述符，可以直接将json字符串写入文件，无返回值
* `json.dumps(obj)` 传入json字典返回json字符串

还有一些关键字参数，可以控制json输出的格式，这里就不尝试了，具体请参考文档。
