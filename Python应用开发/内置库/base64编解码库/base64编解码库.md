# base64编解码库

下面介绍Python的base64模块。

## 字符串的base64编解码

```python
import base64

s = b"hello, world"

b64str = base64.b64encode(s)
print(b64str)

s = base64.b64decode(b64str)
print(s)
```

运行结果：
```
b'aGVsbG8sIHdvcmxk'
b'hello, world'
```

* `base64.b64encode(s)` 传入bytes对象，进行base64编码，结果还是bytes对象
* `base64.b64decode(s)` 传入bytes对象，进行base64解码，结果还是bytes对象

## 对二进制文件进行base64编解码

```python
import base64

fp = open("1.jpg", "rb")
img_bytes = fp.read()
fp.close()

b64_str = base64.b64encode(img_bytes)
print(b64_str)

img_bytes = base64.b64decode(b64_str)
fp = open("2.jpg", "wb")
fp.write(img_bytes)
fp.close()
```

运行结果：
```
b'/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAMCAgMCA...（太长省略）
```

由于`b64encode`和`b64decode`的参数和返回值都是bytes对象，因此使用同样的方法也能将图片变换为base64字符串。
