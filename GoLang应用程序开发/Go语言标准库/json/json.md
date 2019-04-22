# json

Go语言标准库内置了Json序列化和反序列化库，这个包位于`encoding/json`，能够实现将结构体与json字符串之间相互转换。然而，Go语法比较简易，而且是静态类型的，你可能不得不要写一大堆嵌套的结构体。好在Go语言也支持对map进行序列化和反序列化，只不过使用map失去了静态类型语言编译期类型检查的优势。具体用哪个还要自己权衡。

* json.Marshal()：序列化为json
* json.Unmarshal()：反序列化json

## struct序列化为json

```go
package main

import (
	"encoding/json"
	"log"
	"fmt"
)

type RespJson struct {
	Status int    `json:"status"`
	Msg    string `json:"msg"`
}

func main() {
	resp := RespJson{200, "success"}
	jsonStr, err := json.Marshal(&resp)
	if err != nil {
		log.Fatal(err.Error())
	}
	fmt.Println(string(jsonStr))
}
```

上面代码中，在定义结构体时，我们通过反引号标签的形式，指定了序列化为json时，该字段的字段名。

## json反序列化为struct

```go
package main

import (
	"encoding/json"
	"fmt"
)

type RespJson struct {
	Status int    `json:"status"`
	Msg    string `json:"msg"`
}

func main() {
	str := "{\"status\":200,\"msg\":\"success\"}"
	resp := RespJson{}
	json.Unmarshal([]byte(str), &resp)
	fmt.Println(resp)
}
```

## map序列化为json

我们组装json时，其实可以偷懒用map，反序列化时就免了，反序列化还是建议映射到struct上，便于后续操作。

```go
package main

import (
	"encoding/json"
	"log"
	"fmt"
)

func main() {
	m := make(map[string]interface{})
	m["status"] = 200
	m["msg"] = "success"
	b, err := json.Marshal(m)
	if err != nil {
		log.Fatal(err.Error())
	}
	fmt.Println(string(b))
}
```
