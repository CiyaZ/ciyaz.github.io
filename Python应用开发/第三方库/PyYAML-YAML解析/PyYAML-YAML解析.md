# PyYAML

yaml是一种数据交换格式，类似json，常用作配置文件。这里我们看一下如何使用PyYAML读取yaml配置文件。

实际上，PyYAML和Python内置的json库的API非常相似。我们直接看几个例子：

测试使用的yaml
```yaml
name: "Tom"
age: 10
languages:
  - name: "python"
    developer: "Guido van Rossum"
    maintainer: "python.org"
  - name: "java"
    developer: "James Gosling"
    maintainer: "oracle"
  - name: "c"
    developer: "AT&T Bell Laboratory"
```

## 解析yaml字符串

```python
import yaml

fp = open("test.yaml", "r")
dict = yaml.load(fp)
print(dict)
fp.close()
```

* `yaml.load(stream)` 我们可以传入一个文件流对象作为参数，load()会自动解析里面的内容
* `yaml.load(str)` 我们也可以传入字符串。不过既然是读取配置文件嘛，最常用的还是上面文件流作参数

## 序列化字典到yaml字符串

在上面代码基础上加了两行。

```python
str = yaml.dump(dict)
print(str)
```

运行结果：

```yaml
age: 10
languages:
- {developer: Guido van Rossum, maintainer: python.org, name: python}
- {developer: James Gosling, maintainer: oracle, name: java}
- {developer: AT&T Bell Laboratory, name: c}
name: Tom
```

和原来不太一样啊，不过yaml也支持这种写法。这里的内容和原来的内容是等价的。
