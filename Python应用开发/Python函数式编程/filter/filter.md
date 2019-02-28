# filter函数

`filter()`可以用来过滤一个可迭代对象。

例子：

```python
l = filter(lambda x: x % 2 == 0, [1, 2, 3, 4, 5, 6])
print(list(l))
```

输出结果：
```
[2, 4, 6]
```

filter函数的第一个参数是一个函数，传入一个值，返回True或False。filter就是根据这个返回值确定后面列表中元素的去留的。
