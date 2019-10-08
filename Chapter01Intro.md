# Golang Introduction

- [Golang Introduction](#golang-introduction)
	- [Go Features](#go-features)
	- [Go package](#go-package)

## Go Features

1. gc
   1. 只需要new分配内存，不需要释放
   1. 内存自动回收，再也不需要开发人员管理内存
1. 天然并发
   1. 从语言层面支持并发，非常简单
   1. goroutine，轻量级线程(本质是协程映射到物理线程上,但比操作系统的线程轻量)，创建成千上万个goroutine成为可能(而创建成千上万个线程，程序会被OS kill)，能够充分利用多核CPU
   1. 基于CSP（Communicating Sequential Process）模型实现
1. channel
   1. 多个goroutine之间通过channel进行通信, channel类似Unix/Linux下的pipe
   1. channel支持任何数据类型
1. 多返回值

example: simple example

```go
package main

import "fmt"

func add(a int, b int) int {
	var sum int
	sum = a + b
	return sum
}

func main() {
	var s int
	s = add(100, 10)
	// 如果变量定义不使用，不能通过编译
	fmt.Println("Result=", s)
}
```

example: goroutine
> run in terminal: `go run hello.go test.go`  
> run in vscode: `F5`

```bash
./
    hello.go
    test.go
```

```go
// test.go
package main

import "fmt"

func testGoroutine(a int) {
	fmt.Println(a)
}
```

```go
// hello.go, 两个文件同属一个package: main
package main

import (
	"fmt"
	"time"
)

func add(a int, b int) int {
	var sum int
	sum = a + b
	return sum
}

func main() {
	var s int
	s = add(100, 10)
	fmt.Println("Result=", s)

	// 并发打印
	for i := 0; i < 20; i++ {
		go testGoroutine(i)
	}
	// 主rouinte等待 3s
	time.Sleep(time.Second * 3)
}
```

example: 多核CPU占用100%

```go
// test.go
package main

func testInfLoop(a int) {
	for {

	}
}
```

```go
// hello.go
package main

import (
	"time"
)

func main() {
	for i := 0; i < 20; i++ {
		go testInfLoop(i)
	}
	// 主rouinte等待
	time.Sleep(time.Second * 60)
}
```

example: channel
> 全局变量虽然可以作为goroutine的数据交互但不推荐

```go
package main

import "fmt"

func testChannel() {
	// make分配内存空间
	// channel中放的是int, 容量是3
	pipe := make(chan int, 3)
	// 放数据进channel
	pipe <- 1
	pipe <- 11
	pipe <- 111

	// use data in pipe
	a := <-pipe
	b := <-pipe
	pipe <- 6
	fmt.Println(len(pipe)) // 2
	fmt.Println(a, b)      //1 11
}

func main() {
	testChannel()
}
```

```go
var pipe1 chan int
pipe1 = make(chan int, 3)
// 两者等价
pipe2 := make(chan int, 3)

var a int
a = <- pipe2
// 两者等价
a := <- pipe2
```

example: channel 阻塞

```go
package main

import (
	"fmt"
	"time"
)

func testChannel() {
	pipe := make(chan int, 3)
	pipe <- 1
	pipe <- 11
	pipe <- 111
	pipe <- 1111 // 阻塞

	fmt.Println(len(pipe))
}

func main() {
	println("start goroutine")
	go testChannel()
	println("end goroutine")
	time.Sleep(10 * time.Second)
}
```

trick: 格式化代码
- `gofmt test.go`: 格式化test.go文件，并在terminal中显示，文件并没有改变
- `gofmt -w test.go`: 格式化test.go文件，并写入文件

example: 主routine从次routine中获取数据，并且channel是全局变量(不推荐)

```go
package main

import "fmt"

var pipe chan int

func add(a int, b int) {
	sum := a + b
	pipe <- sum
}

func main() {
	pipe = make(chan int, 1)
	go add(100, 200)
	b := <-pipe // wait here
	fmt.Println(b)
}
```

example: 主routine从次routine中获取数据

```go
package main

import "fmt"

func add(a int, b int, p chan int) {
	sum := a + b
	p <- sum
}

func main() {
	var pipe chan int
	pipe = make(chan int, 1)
	go add(100, 200, pipe) // pipe本质是指针
	b := <-pipe
	fmt.Println(b)
}
```

example: 多返回值

```go
package main

import "fmt"

func calc(a int, b int) (int, int) {
	sum := a + b
	avg := sum / 2
	return sum, avg
}

func do(a int, b int) (float32, int) {
	c := float32(a) / float32(b)
	// c : a/b
	d := a * b
	return c, d
}

func main() {
	sum, avg := calc(100, 200)
	fmt.Println(sum, avg) // 300 150
	c, _ := do(100, 200)
	fmt.Println(c) // 0.5
}
```

编译为二进制文件: 当前工作目录下 `go build`

## Go package

- 和python一样，把相同功能的代码放到一个目录，称之为包
- 包可以被其他包引用
- main包是用来生成可执行文件，每个程序只有一个main包
- 包的主要用途是提高代码的可复用性

```bash
# golang标准package目录
# GOPATH
./
    src/
        mypackage/
            calc/
                add.go
                sub.go
            main/
                main.go
```

```go
// add.go
package calc

// Add is doing adding
func Add(a int, b int) int {
	return a + b
}
```

```go
//sub.go
package calc

// Sub is doing subtract
// 大写的Sub才是public的函数
func Sub(a int, b int) int {
	return a - b
}
```

```go
// main.go
package main

import (
	"fmt"
	"mypackage/calc"
)

func main() {
	sum := calc.Add(100, 200)
	fmt.Println(sum)
	sub := calc.Sub(100, 200)
	fmt.Println(sub)
}
```

build: `go build mypackage/main`

example: goroutine package

```bash
src/
    myroutine/
        calc/
            add.go
        main/
            main.go
```

```go
// add.go
package calc

// Add is a gorouine method
func Add(a int, b int, p chan int) {
	c := a + b
	p <- c
}
```

```go
// main.go
package main

import (
	"fmt"
	"myroutine/calc"
)

func main() {
	pipe := make(chan int, 3)
	go calc.Add(100, 200, pipe)
	sum := <-pipe
	fmt.Println(sum)
}
```

example: simple string format

```go
package main

import "fmt"

func main() {
	fmt.Printf("hello, grey\n")
	fmt.Printf("%b\n", 10)
	fmt.Printf("%d\n", 10)
	fmt.Printf("%x\n", 10)
	fmt.Printf("%.2f", 12.36)
}
```