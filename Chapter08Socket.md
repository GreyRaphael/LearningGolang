# Golang Socket

- [Golang Socket](#golang-socket)
	- [tcp](#tcp)
	- [redis with golang](#redis-with-golang)
	- [http](#http)
	- [template](#template)
	- [MySQL with golang](#mysql-with-golang)

## tcp

Server:
- 监听端口
- 接收客户端的链接
- 创建goroutine，处理该链接

Client:
- 建立与服务端的链接
- 进行数据收发
- 关闭链接

example: tcp server

```go
package main

import (
	"fmt"
	"net"
)

func main() {
	fmt.Println("start server...")
	listen, err := net.Listen("tcp", "0.0.0.0:6666")
	if err != nil {
		fmt.Println("listen failed, err:", err)
		return
	}
	for {
		conn, err := listen.Accept()
		if err != nil {
			fmt.Println("accept failed, err:", err)
			continue
		}
		go process(conn)
	}
}

func process(conn net.Conn) {
	defer conn.Close()

	fmt.Printf("Serving %s\n", conn.RemoteAddr().String())
	for {
		buf := make([]byte, 1024)
		n, err := conn.Read(buf) // 也可以bufio.NewReader(buff).ReadString('\n')
		if err != nil {
			fmt.Println("read err:", err)
			return
		}
		fmt.Printf("Received: %v\n", string(buf[:n]))
	}
}
```

example: tcp client

```go
package main

import (
	"bufio"
	"fmt"
	"net"
	"os"
	"strings"
)

func main() {
	conn, err := net.Dial("tcp", "162.105.54.11:8080")
	if err != nil {
		fmt.Println("Error dialing", err.Error())
		return
	}
	defer conn.Close()

	inputReader := bufio.NewReader(os.Stdin)
	for {
		input, _ := inputReader.ReadString('\n')
		trimmedInput := strings.Trim(input, "\r\n")
		if trimmedInput == "Q" {
			return
		}
		_, err = conn.Write([]byte(trimmedInput))
		if err != nil {
			return
		}
	}
}
```

example: 发送http请求
> 返回既包含Respone Headers又包含Response Body

```go
package main

import (
	"fmt"
	"io"
	"net"
)

func main() {
	conn, err := net.Dial("tcp", "www.baidu.com:80")
	if err != nil {
		fmt.Println("Error dialing", err.Error())
		return
	}
	defer conn.Close()

	msg := "GET / HTTP/1.1\r\n"
	msg += "Host: www.baidu.com\r\n"
	msg += "Connection: close\r\n"
	msg += "\r\n\r\n"

	_, err = io.WriteString(conn, msg)
	if err != nil {
		fmt.Println("write string failed, ", err)
		return
	}
	buf := make([]byte, 4096)
	for {
		count, err := conn.Read(buf)
		if err != nil {
			break
		}
		fmt.Println(string(buf[0:count]))
	}
}
// go build
// ./main.ext > text.html
```

## redis with golang

- redis是个开源的高性能的key-value的内存数据库，可以把它当成远程的数据结构。
- 支持的value类型非常多，比如string、list（链表）、set（集合）、
hash表等等
- redis性能非常高，单机能够达到15w qps，通常适合做缓存。

[3rd go-redis](https://github.com/go-redis/redis): `go get -u github.com/go-redis/redis/v7`

example: redis `Set`, `Get`

```go
package main

import (
	"fmt"

	"github.com/go-redis/redis/v7"
)

func main() {
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // no password set
		DB:       0,  //use default DB
	})
	defer client.Close()

	err := client.Set("key", 100, 0).Err()
	if err != nil {
		panic(err)
	}

	val, err := client.Get("key").Result()
	if err != nil {
		panic(err)
	}
	fmt.Printf("%T, %#v\n", val, val) // string, "100"

	val2, err := client.Get("key2").Result()
	if err == redis.Nil {
		fmt.Println("key2 does not exist")
	} else if err != nil {
		panic(err)
	} else {
		fmt.Println("key2", val2)
	}
}
```

example: redis `MSet`, `MGet`

```go
package main

import (
	"fmt"

	"github.com/go-redis/redis/v7"
)

func main() {
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // no password set
		DB:       0,  //use default DB
	})
	defer client.Close()

	err := client.MSet("name", "grey", "age", 22).Err()
	if err != nil {
		panic(err)
	}

	val, err := client.MGet("name", "age").Result()
	if err != nil {
		panic(err)
	}
	fmt.Printf("%T, %#v\n", val, val) // []interface {}, []interface {}{"grey", "22"}
}
```

example: redis `expire`

```go
package main

import (
	"fmt"
	"time"

	"github.com/go-redis/redis/v7"
)

func main() {
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // no password set
		DB:       0,  //use default DB
	})
	defer client.Close()

	err := client.MSet("name", "grey", "age", 22).Err()
	if err != nil {
		panic(err)
	}

	val, err := client.Expire("name", 10*time.Second).Result()
	if err != nil {
		panic(err)
	}
	fmt.Printf("%T, %#v\n", val, val) // bool, true
}
```

exmaple: redis `HSet`, `HGet`, `HGetAll`

```go
package main

import (
	"fmt"

	"github.com/go-redis/redis/v7"
)

func main() {
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // no password set
		DB:       0,  //use default DB
	})
	defer client.Close()

	err := client.HSet("student1", "name", "chris").Err()
	if err != nil {
		panic(err)
	}
	err = client.HSet("student1", "age", 66).Err()
	if err != nil {
		panic(err)
	}

	val, err := client.HGet("student1", "name").Result()
	if err != nil {
		panic(err)
	}
	fmt.Printf("%T, %#v\n", val, val) // string "chris"
}
```

example: redis `HMSet`, `HMGet`

```go
package main

import (
	"fmt"

	"github.com/go-redis/redis/v7"
)

func main() {
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // no password set
		DB:       0,  //use default DB
	})
	defer client.Close()

	// HSet, HMSet用于存object
	err := client.HMSet("student1", map[string]interface{}{"name": "chris", "age": 66}).Err()
	if err != nil {
		panic(err)
	}

	val, err := client.HMGet("student1", "name", "age").Result()
	if err != nil {
		panic(err)
	}
	fmt.Printf("%T, %#v\n", val, val) // []interface {}, []interface {}{"chris", "66"}
}
```

example: redis list `LPush`, `LPop`

```go
package main

import (
	"fmt"

	"github.com/go-redis/redis/v7"
)

func main() {
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // no password set
		DB:       0,  //use default DB
	})
	defer client.Close()

	err := client.LPush("students", "stu1", "stu2", "stu3").Err()
	if err != nil {
		panic(err)
	}

	val, err := client.LPop("students").Result()
	if err != nil {
		panic(err)
	}
	fmt.Printf("%T, %#v\n", val, val) // string, "stu3"
}
```

## http

- Go原生支持http，`import("net/http")`
- Go的http服务性能和nginx比较接近, 那么就可以少用一个nginx了(可能有多级nginx架构))
- 几行代码就可以实现一个web服务

example: http server

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func greet(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello World! %s", time.Now())
}

func login(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "this is login page")
}

func main() {
	http.HandleFunc("/", greet)
	http.HandleFunc("/login", login)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("http listen failed")
	}
	// test following urls
	// localhost:8080/
	// localhost:8080/xxx
	// localhost:8080/login
	// localhost:8080/login/xxx
}
```

http request method:
- get: in url, <8k
- post: in body, no limit
- put
- delete
- head

example: get request

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	resp, err := http.Get("https://httpbin.org/get")
	if err != nil {
		panic(err)
	}

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}

	fmt.Println(string(body))
}
```

example: post requst

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	jsonData, err := json.Marshal(map[string]interface{}{
		"name": "grey",
		"age":  66,
	})
	if err != nil {
		panic(err)
	}

	resp, err := http.Post("https://httpbin.org/post", "application/json", bytes.NewBuffer(jsonData))
	if err != nil {
		panic(err)
	}

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}

	fmt.Println(string(body))
}
```

example: custom post request

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
	"time"
)

func main() {
	jsonData, err := json.Marshal(map[string]interface{}{
		"name": "grey",
		"age":  66,
	})
	if err != nil {
		panic(err)
	}

	client := http.Client{Timeout: time.Duration(3 * time.Second)}
	request, err := http.NewRequest("POST", "https://httpbin.org/post", bytes.NewBuffer(jsonData))
	request.Header.Set("Content-type", "application/json")
	if err != nil {
		panic(err)
	}

	resp, err := client.Do(request)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}

	fmt.Println(string(body))
}
```

example: `postForm`

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"net/url"
)

func main() {
	formData := url.Values{
		"name": {"grey", "moris"},
		"age":  {"66"},
	}

	resp, err := http.PostForm("https://httpbin.org/post", formData)
	if err != nil {
		panic(err)
	}

	// body, err := ioutil.ReadAll(resp.Body)
	// if err != nil {
	// 	panic(err)
	// }

	// fmt.Println(string(body))

	// decode data
	result := map[string]interface{}{}
	json.NewDecoder(resp.Body).Decode(&result)
	fmt.Println(result["form"])
}
```

example: head request

```go
package main

import (
	"fmt"
	"net/http"
)

var url = []string{
	"http://www.baidu.com",
	"http://google.com",
	"http://taobao.com",
}

func main() {

	for _, v := range url {
		resp, err := http.Head(v)
		if err != nil {
			fmt.Printf("head %s failed, err:%v\n", v, err)
			continue
		}

		fmt.Printf("head succeed, status:%v\n", resp.Status)
	}
}
```

example: custom head 请求 with timeout

```go
package main

import (
	"fmt"
	"net"
	"net/http"
	"time"
)

var url = []string{
	"http://www.baidu.com",
	"http://google.com",
	"http://taobao.com",
}

func main() {

	for _, v := range url {
		client := http.Client{
			Transport: &http.Transport{
				Dial: func(network, addr string) (net.Conn, error) {
					return net.DialTimeout(network, addr, 3*time.Second)
				},
			},
		}

		resp, err := client.Head(v)
		if err != nil {
			fmt.Printf("head %s failed, err:%v\n", v, err)
			continue
		}

		fmt.Printf("head succeed, status:%v\n", resp.Status)
	}
}
```

example: head request with timeout simple

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

var url = []string{
	"http://www.baidu.com",
	"http://google.com",
	"http://taobao.com",
}

func main() {

	for _, v := range url {
		client := http.Client{
			Timeout: time.Duration(3 * time.Second),
		}

		resp, err := client.Head(v)
		if err != nil {
			fmt.Printf("head %s failed, err:%v\n", v, err)
			continue
		}

		fmt.Printf("head succeed, status:%v\n", resp.Status)
	}
}
```

常见状态码:
- `http.StatusContinue = 100`: 服务器同意上传的时候返回100
- `http.StatusOK = 200`
- `http.StatusFound = 302`: 跳转
- `http.StatusBadRequest = 400`: 非法请求，协议包有问题，服务器无法解析
- `http.StatusUnauthorized = 401`: 权限无法通过
- `http.StatusForbidden = 403`: 禁止访问
- `http.StatusNotFound = 404`
- `http.StatusInternalServerError = 500`
- 502: nginx请求php, php挂了，返回502

example: server处理form
> 一般都是browser发送json, 然后golang server处理json, 最后返回broswer json;  
> 这种场景已经很少了

```go
package main

import (
	"io"
	"net/http"
)

const form = `<html><body><form action="#" method="post" name="bar">
                    <input type="text" name="in"/>
                    <input type="text" name="in"/>
                     <input type="submit" value="Submit"/>
             </form></html></body>`

func SimpleServer(w http.ResponseWriter, request *http.Request) {
	io.WriteString(w, "<h1>hello, world</h1>")
}

func FormServer(w http.ResponseWriter, request *http.Request) {
	w.Header().Set("Content-Type", "text/html")
	switch request.Method {
	case "GET":
		io.WriteString(w, form)
	case "POST":
		request.ParseForm()
		io.WriteString(w, request.Form["in"][0])
		io.WriteString(w, "\n")
		io.WriteString(w, request.FormValue("in"))
	}
}
func main() {
	http.HandleFunc("/test1", SimpleServer)
	http.HandleFunc("/test2", FormServer)
	if err := http.ListenAndServe(":8088", nil); err != nil {
	}
}
```

example: 所有的处理函数都加上panic处理

```go
package main

import (
	"io"
	"log"
	"net/http"
)

const form = `<html><body><form action="#" method="post" name="bar">
                    <input type="text" name="in"/>
                    <input type="text" name="in"/>
                     <input type="submit" value="Submit"/>
             </form></html></body>`

func main() {
	http.HandleFunc("/test1", logPanics(SimpleServer))
	http.HandleFunc("/test2", logPanics(FormServer))
	if err := http.ListenAndServe(":8088", nil); err != nil {
	}
}

func logPanics(handle http.HandlerFunc) http.HandlerFunc {
	return func(writer http.ResponseWriter, request *http.Request) {
		defer func() {
			if x := recover(); x != nil {
				log.Printf("[%v] caught panic: %v", request.RemoteAddr, x)
			}
		}()
		handle(writer, request)
	}
}

func SimpleServer(w http.ResponseWriter, request *http.Request) {
	io.WriteString(w, "<h1>hello, world</h1>")
	panic("this is a panic")
}

func FormServer(w http.ResponseWriter, request *http.Request) {
	w.Header().Set("Content-Type", "text/html")
	switch request.Method {
	case "GET":
		io.WriteString(w, form)
	case "POST":
		request.ParseForm()
		io.WriteString(w, request.Form["in"][0])
		io.WriteString(w, "\n")
		io.WriteString(w, request.FormValue("in"))
	}
}
```

## template

```bash
src/
	project1/
		main/
			main.go
			index.html
```

```html
<!-- index.html -->
<html>

<head>
    <title>Document</title>
</head>

<body>
    <p> hello, {{.Name}}</p>
    <p> {{.Age}}</p>
</body>

</html>
```

```go
// main.go
package main

import (
	"fmt"
	"net/http"
	"text/template"
)

type Person struct {
	Name string
	Age  int
}

type myWriter struct {
	data string
}

func (m *myWriter) Write(p []byte) (n int, err error) {
	// implement io.Writer interface
	m.data += string(p)
	return len(p), nil
}

func SimpleServer(w http.ResponseWriter, request *http.Request) {
	t, err := template.ParseFiles("index.html")
	if err != nil {
		fmt.Println("parse file err")
		return
	}

	p := &Person{"grey", 22}
	if err = t.Execute(w, p); err != nil { // 替换template中的{{.FieldName}}
		fmt.Println("execute error")
	}

	// dump to custom struct
	myw := &myWriter{}
	err = t.Execute(myw, p)
	fmt.Printf("%#v\n", myw) // html string
}

func main() {
	http.HandleFunc("/temp1", SimpleServer)
	if err := http.ListenAndServe(":8088", nil); err != nil {
		fmt.Println("server crash")
	}
}
```

```go
// print to console
package main

import (
	"fmt"
	"os"
	"text/template"
)

type Person struct {
	Name string
	Age  int
}

func main() {
	t, err := template.ParseFiles("index.html")
	if err != nil {
		fmt.Println("parse file err:", err)
		return
	}

	p := Person{"grey", 33}
	if err := t.Execute(os.Stdout, p); err != nil {
		fmt.Println("There was an error:", err.Error())
	}
}
```

example: template with `if`

```html
<!-- index.html -->
<html>

<head>
</head>

<body>
    This is:{{.}}
	<!-- {{.}}表示这个对象 -->
    {{if gt .Age 18}}
    <p>hello, old man, {{.Name}}</p>
    {{else}}
    <p>hello,young man, {{.Name}}</p>
    {{end}}
</body>

</html>
```

- not 非`{{if not .condition}} {{end}}`
- and 与`{{if and .condition1 .condition2}} {{end}}`
- or 或`{{if or .condition1 .condition2}} {{end}}`
- eq 等于`{{if eq .var1 .var2}} {{end}}`
- ne 不等于`{{if ne .var1 .var2}} {{end}}`
- lt 小于 `{{if lt .var1 .var2}} {{end}}`
- le 小于等于`{{if le .var1 .var2}} {{end}}`
- gt 大于`{{if gt .var1 .var2}} {{end}}`
- ge 大于等于`{{if ge .var1 .var2}} {{end}}`

example: `{{with .Var}}   {{end}}`
> with里面的`{{.}}`就是Var的值

```html
<!-- index.html -->
<html>

<head>
</head>

<body>
    {{with .Name}}
    <p>hello, old man, {{.}}</p>
    {{end}}
</body>

</html>
```

example: `{{range}} {{end}}`

```go
// main.go
func SimpleServer(w http.ResponseWriter, request *http.Request) {
	t, err := template.ParseFiles("index.html")
	if err != nil {
		fmt.Println("parse file err")
		return
	}

	// many persons
	p := []Person{Person{"grey", 16}, Person{"alpha", 22}}
	if err = t.Execute(w, p); err != nil { 
		fmt.Println("execute error")
	}
}
```

```html
<!-- index.html -->
<html>

<head>
</head>

<body>
    {{range .}}
    {{if gt .Age 18}}
    <p>hello, old man, {{.Name}}</p>
    {{else}}
    <p>hello,young man, {{.Name}}</p>
    {{end}}
    {{end}}
</body>

</html>
```

## MySQL with golang

prequisites:
- `go get github.com/jmoiron/sqlx`
- `go get -u github.com/go-sql-driver/mysql`

example: initialize 2 tables

```sql
CREATE TABLE person (
    user_id int primary key auto_increment,
    username varchar(260),
    sex varchar(260),
    email varchar(260)
);

CREATE TABLE place (
    country varchar(200),
    city varchar(200),
    telcode int
)
```

example: simple insert data

```go
package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

func main() {
	db, err := sqlx.Open("mysql", "root:password@tcp(localhost:3306)/Test")
	if err != nil {
		fmt.Println("connect db error", err)
		return
	}

	// insert
	for i := 0; i < 6; i++ {
		_, err := db.Exec("insert into person(username, sex, email)values(?,?,?)", fmt.Sprintf("Stu%d", i), "male", fmt.Sprintf("stu%d@qq.com", i))
		if err != nil {
			fmt.Println("insert error")
			return
		}
	}
}
```

exmaple: insert

```go
package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

var Db *sqlx.DB

func init() {
	db, err := sqlx.Open("mysql", "root:password@tcp(localhost:3306)/Test")
	if err != nil {
		fmt.Println("connect db error", err)
		return
	}
	Db = db
}

func main() {
	r, err := Db.Exec("insert into person(username, sex, email)values(?,?,?)", "jane", "femlae", "jane@qq.com")
	if err != nil {
		fmt.Println("insert error")
		return
	}

	id, err := r.LastInsertId()
	if err != nil {
		fmt.Println("exec failed", err)
		return
	}

	fmt.Println("insert success:", id)
}
```

example: select

```go
package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

type Person struct {
	UserID   int    `db:"user_id"`
	Username string `db:"username"`
	Sex      string `db:"sex"`
	Email    string `db:"email"`
}

var Db *sqlx.DB

func init() {
	db, err := sqlx.Open("mysql", "root:password@tcp(localhost:3306)/Test")
	if err != nil {
		fmt.Println("connect db error", err)
		return
	}
	Db = db
}

func main() {
	var p []Person // must be a slice
	err := Db.Select(&p, "select user_id, username, sex, email from person where user_id=?", 11)
	if err != nil {
		fmt.Println("select error", err)
		return
	}

	fmt.Println("select success:", p) // select success: [{11 Stu4 male stu4@qq.com}]
}
```

example: update

```go
func main() {
	r, err := Db.Exec("update person set username=? where user_id=?", "moris", 10)
	if err != nil {
		fmt.Println("update error", err)
		return
	}

	num, err := r.RowsAffected()
	if err != nil {
		fmt.Println("update error", err)
		return
	}
	fmt.Println("updte", num) // update 1
}
```

example: delete

```go
func main() {
	r, err := Db.Exec("delete from person where user_id=?", 9)
	if err != nil {
		fmt.Println("delete error", err)
		return
	}

	num, err := r.RowsAffected()
	if err != nil {
		fmt.Println("delete error", err)
		return
	}
	fmt.Println("delete", num) // delete 1
}
```