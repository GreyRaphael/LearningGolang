# Golang Socket

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

