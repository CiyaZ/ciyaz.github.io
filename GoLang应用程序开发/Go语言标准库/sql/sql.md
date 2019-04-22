# sql

Go语言内置了一个通用的数据库操作接口，位于`database/sql`包，但标准库没有具体的实现和数据库驱动，这和Java有JDBC，同时需要单独加载数据库驱动一样。JDBC接口的设计其实滥用了受检查异常，导致代码异常难写，而且因为历史原因，也就只能那样了，Go语言稍微好点。

我们这里以MySQL为例进行介绍，网上大多数使用Go语言连接MySQL，都是使用这个第三方Go语言驱动，项目地址：[https://github.com/go-sql-driver/mysql/](https://github.com/go-sql-driver/mysql/)

该项目开源协议是`MPL2.0`，使用时要注意一下，另外重要的上线项目要对这些看起来不是太靠谱的库进行充分的代码审计，不要留下安全隐患。

我们直接使用`go get`下载下来使用：

```
go get github.com/go-sql-driver/mysql
```

在代码中引入：`import _ "github.com/go-sql-driver/mysql"`

## 连接数据库

```go
db, err := sql.Open("mysql", "root:root@tcp(127.0.0.1:3306)/netstore")
if err != nil {
  log.Fatal(err.Error())
}
```

Go语言的连接字符串和JDBC不同，格式为`用户名:密码@tcp(主机:端口)/数据库名`。

## 查询一条数据

查询一条数据可以使用`QueryRow`实现，下面代码中，对应表结构定义了一个结构体，查询结果被放入结构体中。

```go
package main

import (
	"database/sql"
	"log"
	"fmt"
  _ "github.com/go-sql-driver/mysql"
)

type Cust struct {
	CustId   uint64
	CustName string
	AreaId   uint64
	age      uint32
	sex      string
	tel      string
	email    string
	address  string
}

func main() {
	db, err := sql.Open("mysql", "root:root@tcp(127.0.0.1:3306)/netstore")
	if err != nil {
		log.Fatal(err.Error())
	}

	id := 1
	cust := Cust{}
	err = db.QueryRow("select * from t_cust where cust_id=?", id).Scan(
		&cust.CustId,
		&cust.CustName,
		&cust.AreaId,
		&cust.age,
		&cust.sex,
		&cust.tel,
		&cust.email,
		&cust.address)
	if err != nil {
		log.Fatal(err.Error())
	}

	fmt.Println(cust)

}
```

注：代码只是个例子，千万不要在正式场合写`select *`，一定要把字段写全，因为这个问题不知被坑了多少次了

## 查询多条数据

查询多条结果时，使用`Query`函数，返回值为`Rows`，我们通过`Next()`函数遍历查询结果集，取出每条记录并打印。

```go
package main

import (
	"database/sql"
	"log"
	"fmt"
  _ "github.com/go-sql-driver/mysql"
)

type Cust struct {
	CustId   uint64
	CustName string
	AreaId   uint64
	age      uint32
	sex      string
	tel      string
	email    string
	address  string
}

func main() {
	db, err := sql.Open("mysql", "root:root@tcp(127.0.0.1:3306)/netstore")
	if err != nil {
		log.Fatal(err.Error())
	}

	rows, err := db.Query("select * from t_cust limit 0, 10")
	if err != nil {
		log.Fatal(err.Error())
	}
	for rows.Next() {
		cust := Cust{}
		rows.Scan(
			&cust.CustId,
			&cust.CustName,
			&cust.AreaId,
			&cust.age,
			&cust.sex,
			&cust.tel,
			&cust.email,
			&cust.address)
		fmt.Println(cust)
	}

}
```

## 增删改数据

增删改写法都是一样的，都是执行一条预编译SQL语句即可。这里以增加一条数据为例。

```go
package main

import (
	"database/sql"
	"log"
	_ "github.com/go-sql-driver/mysql"
	"fmt"
)

func main() {
	db, err := sql.Open("mysql", "root:root@tcp(127.0.0.1:3306)/netstore")
	if err != nil {
		log.Fatal(err.Error())
	}

	tx, err := db.Begin()
	if err != nil {
		log.Fatal(err.Error())
	}

	stmt, err := tx.Prepare("insert into t_product (product_name, type_id, cost, price) values (?, ?, ?, ?)")
	if err != nil {
		log.Fatal(err.Error())
	}

	res, err := stmt.Exec("强力卫生巾", 1, 5.72, 19.9)
	if err != nil {
		log.Fatal(err.Error())
	}

	err = tx.Commit()
	if err != nil {
		log.Fatal(err.Error())
	}

	newId, err := res.LastInsertId()
	fmt.Println(newId)
}
```

这个错误处理就非常讨厌了，以上步骤，只要有一步出错，后续步骤就无法进行下去，因此其他语言可以使用`try...catch...`统一抓异常，Go语言就不行。不过毕竟Go语言解决了一些痛点，凑合着用也行了。
