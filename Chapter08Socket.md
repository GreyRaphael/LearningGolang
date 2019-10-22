# Golang Socket

- [Golang Socket](#golang-socket)
	- [tcp](#tcp)
	- [redis with golang](#redis-with-golang)
	- [http](#http)

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

example: http client
> 本质就是浏览器, 只能处理http不能处理https

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	res, err := http.Get("http://www.baidu.com/")
	if err != nil {
		fmt.Println("get err:", err)
		return
	}

	data, err := ioutil.ReadAll(res.Body)
	if err != nil {
		fmt.Println("get data err:", err)
		return
	}

	fmt.Println(string(data))
}
```

http request method:
- get: in url, <8k
- post: in body, no limit
- put
- delete
- head

example: head 请求

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
> 一般都是json来请求，这种场景几乎没有

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