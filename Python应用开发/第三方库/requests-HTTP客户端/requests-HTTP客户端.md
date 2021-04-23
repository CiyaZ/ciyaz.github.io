# Requests

Requests是一个辅助http操作的函数库，原本python内置了`urllib`，`urllib2`，`urllib3`也能完成同样的功能，但API设计的都不是很好用。Request则是一个相当简单易用的函数库，使用前`import requests`就行了，爬虫，web接口调用等程序都会用到。由于Requests实在太好用了，导致其他语言也纷纷移植，比如Java的Requests库。

使用Requests前，我们熟练掌握HTTP协议。Requests库的使用十分简单，下面主要以例子的形式介绍其中的函数。

## 发送请求和接受响应

### 请求网页

GET请求HTML例子：

```python
import requests

params = {'wd': '美女'}

r = requests.get("http://www.baidu.com/s", params=params)
r.encoding = "utf-8"
print(r.text)
```

* `requests.get()`会发起GET请求，返回值是一个响应（response）对象，`params`为URL参数
* `r.encoding`可以更改读取返回值时使用的文本编码
* `r.text`读取响应的文本内容

同理，HTTP的post，delete，head，options的请求使用也是类似的。

### 请求图片

请求二进制数据的例子：

```python
r = requests.get("https://www.baidu.com/img/bd_logo1.png")

fp = open("test.png", "wb")
fp.write(r.content)
fp.close()
```

我们请求了一个图片并保存到文件。

### 请求json

还有一种情况是请求json，这通常用于web接口调用。

```python
r = requests.get("http://www.weather.com.cn/data/sk/101010100.html")
r.encoding="utf-8"
print(r.json())
```

上面例子是查询一个天气接口。

Requests内置了一个json模块，可以对返回的json字符串进行解析。

### 读取响应码和响应头

```python
r = requests.get("https://www.baidu.com")
print(r.status_code)
print(r.headers)
```

* `r.status_code` 是响应对象的响应码
* `r.headers` 是响应对象的响应头，是一个字典对象，我们可以按照字典的规则，按键值对取用其中的内容

### 设置请求的cookie

请求时使用cookie，只需设置下`cookies`关键字参数，是一个字典类型。

```python
cookies = {"username": "test"}
r = requests.get(url, cookies=cookies)
```

### 读取响应cookie

* `r.cookies`可以读取服务器设置的`cookie`，是一个字典类型。

## 传递URL参数

我们不必自己拼http请求字符串。

调用百度进行关键字搜索：
```python
params = {"wd": "python3"}
r = requests.get("http://www.baidu.com/s", params=params)
```

传入一个关键字参数`params`即可实现URL参数，它的类型是一个字典。

* 我们可以使用`r.url`查看拼好的请求URL字符串

## 设置请求头

有些服务器会主动判断`User-Agent`，根据终端设备的不同返回不同的内容，我们可以使用`headers`关键字参数定制请求头。

```python
headers = {"user-agent":"Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:53.0) Gecko/20100101 Firefox/53.0"}
r = requests.get("http://www.baidu.com", headers=headers)
```

当然不止`User-Agent`，HTTP请求头中具有的参数都可以传。

## POST携带数据发起请求

### 提交表单

我们使用`data`关键字参数进行表单数据的组织。

```python
data = {"key1":"value1", "key2":"value2"}
r = requests.post(url, data=data)
```

### 提交json字符串

直接使用`json`关键字参数就好了。字典会自动转换成json字符串。

```python
data = {"key1":"value1", "key2":"value2"}
r = requests.post(url, json=data)
```

### 上传二进制文件

上传文件使用`files`关键字参数就好了，这里要注意，一定要以二进制形式打开文件。

```python
fp = open("test.png", "rb")
files = {"file": fp}
r = requests.post(url, files=files)
```

## 设置请求超时

* 设置`timeout`关键字参数可以设置超时时间，单位是秒。

## 重定向

除了HEAD，Request会自动处理重定向。
