# Golang Socket

- [Golang Socket](#golang-socket)
	- [tcp](#tcp)
	- [redis with golang](#redis-with-golang)

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

