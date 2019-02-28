# Cookie和Session

很简单，和PHP差不多。

设置session
```python
request.session["is_login"] = True
```

访问session
```python
is_login = request.session["is_login"]
```

设置cookie
```python
def set_cookie(request):
	res = HttpResponse()
	res.set_cookie("key", "cookie")
	return res
```

访问cookie
```python
def get_cookie(request):
	print(request.COOKIES["key"])
	return HttpResponse()
```
