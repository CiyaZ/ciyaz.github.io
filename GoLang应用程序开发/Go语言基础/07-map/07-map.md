# map

map结构在大多数编程语言中都语法内置或是提供了标准库，而且有各种各样的叫法，比如：字典、关联数组、键值对映射，都是一样的意思。Go语言语法内置了这个功能，使用非常方便。

## 创建一个map

下面代码初始化了一个map并赋值：
```go
var m  = make(map[string]Vector2)
m["point1"] = Vector2{0, 0}
m["point2"] = Vector2{1, 1}
```

我们也可以直接指定map的初始值：
```go
var m = map[string]string{"key1" : "key1", "key2" : "key2", "key3" : "key3"}
```

声明一个空的map其值为`nil`，和数组可以`append()`不同，空的map并没有什么用，它不能被设置键和值：
```go
var m map[string]Vector2
```

## map操作

### 添加元素

添加比较简单，前面代码已经实现了，例子：

```go
m["point1"] = Vector2{0, 0}
```

### 删除元素

删除元素可以使用`delete`函数实现：

```go
var m = map[string]string{"key1" : "key1", "key2" : "key2", "key3" : "key3"}
delete(m, "key1")
```

### 获取元素

获取元素直接使用键进行访问即可：

```go
var m = map[string]string{"key1" : "key1", "key2" : "key2", "key3" : "key3"}
fmt.Println(m["key1"])
```

### 检测某个键是否存在

下面写法中，为map的取元素赋值语法多加一个接收变量，能够检查一个键是否在map中存在，`exist`为布尔类型，可用于检查该元素是否存在。

```go
var m = map[string]string{"key1" : "key1", "key2" : "key2", "key3" : "key3"}
ele,exist := m["key1"]
fmt.Println(ele, exist)
```
