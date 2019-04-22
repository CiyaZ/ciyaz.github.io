# http

Go语言内置了HTTP的服务端和客户端，这个包位于`net/http`，我们可以很方便的在其基础上进行开发。

## HTTP服务端

Go的HTTP服务模块使用起来和Java非常像，定义一个处理函数，然后注册下路由就可以了。

### 基本使用

```go
package main

import (
	"net/http"
	"io"
)

func helloHandler(rspw http.ResponseWriter, req *http.Request) {
	rsp := "<h1>It works!</h1>"
	io.WriteString(rspw, rsp)
}

func main() {
	http.HandleFunc("/hello", helloHandler)
	http.ListenAndServe(":80", nil)
}
```

### 注册多个路由

上面写法我们只注册了一个路由，当然也支持注册多个路由。

```go
package main

import (
	"net/http"
	"io"
	"fmt"
)

func helloHandler(rspw http.ResponseWriter, req *http.Request) {
	rsp := "<h1>It works!</h1>"
	io.WriteString(rspw, rsp)
}

func handler1(rspw http.ResponseWriter, req *http.Request) {
	rsp := "aaa"
	io.WriteString(rspw, rsp)
}

func handler2(rspw http.ResponseWriter, req *http.Request) {
	rsp := "bbb"
	io.WriteString(rspw, rsp)
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/hello", helloHandler)
	mux.HandleFunc("/aaa", handler1)
	mux.HandleFunc("/bbb", handler2)
	http.ListenAndServe(":80", mux)
}
```

### 解析请求参数

下面代码演示了如何解析请求参数：

```go
func helloHandler(rspw http.ResponseWriter, req *http.Request) {
	req.ParseForm()
  id := req.Form.Get("id")
	...
}
```

我们知道HTTP协议中请求表单有几种不同的格式（enctype），最常用的格式是`x-www-form-urlencoded`，`ParseForm()`函数就是用于对该类型请求进行解析。

REST接口开发时，有路径参数的概念，我们要想获取路径参数，就得手动用`req.URL.Path`去解析了，我们可以用正则，或是其它的字符串处理手段。

## HTTP客户端

Go语言内置的HTTP客户端也是很容易用的，至少比Java的好太多，和Python的Requests库相比还是稍微差了点，基本就是毁在Go语言这糟糕的异常处理设计上了。

```go
package main

import (
	"net/http"
	"log"
	"io/ioutil"
	"fmt"
)

func main() {
	resp, err := http.Get("http://localhost/hello?id=1")
	if err != nil {
		log.Fatal(err.Error())
	}
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err.Error())
	}
	fmt.Println(string(b))
	defer resp.Body.Close()
}
```

其它HTTP请求方法这里就不多做介绍了，原理都差不多，只不过是各种语言API略有区别，实际使用时照着文档抄就行了。
