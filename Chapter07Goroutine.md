# Golang goroutine

- [Golang goroutine](#golang-goroutine)
	- [channel traverse](#channel-traverse)
	- [others](#others)
	- [unit test](#unit-test)

Process vs Thread vs Coroutine:
- 进程是程序在操作系统中的一次执行过程，系统进行资源分配和调度的一个独立单位
- 线程是进程的一个执行实体，是CPU调度和分派的基本单位，是比进程更小的能独立运行的基本单位
- 一个进程可以创建和销毁多个线程；同一个进程中的多个线程之间可以并发执行
- 协程：独立的栈空间，共享堆空间，调度由用户控制，类似用户级线程；一个线程上可以跑多个协程

## channel traverse

Method1: channel简单遍历
- 读次数<=写次数: 正常
- 读次数>写次数: 
  - 无`close(ch)`: 堵塞
  - 有`close(ch)`: 多出的次数获取的值为0


```go
// 读次数<=写次数
package main

import "fmt"

func main() {
	ch := make(chan int, 20)
	N := 10

	go func() {
		for i := 0; i < N; i++ {
			ch <- i
		}
		// close(ch) //有无close(ch)无影响
	}()

	for i := 0; i < N; i++ {
		fmt.Printf("%v, ", <-ch)
	} //0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
}
```

```go
// 读次数>写次数， 无close(ch)
package main

import "fmt"

func main() {
	ch := make(chan int, 20)
	N := 10

	go func() {
		for i := 0; i < N; i++ {
			ch <- i
		}
	}()

	for i := 0; i < 2*N; i++ {
		fmt.Printf("%v, ", <-ch)
	} //堵塞
}
```

```go
// 读次数>写次数， 有close(ch)
package main

import "fmt"

func main() {
	ch := make(chan int, 20)
	N := 10

	go func() {
		for i := 0; i < N; i++ {
			ch <- i
		}
		close(ch)
	}()

	for i := 0; i < 2*N; i++ {
		fmt.Printf("%v, ", <-ch)
	} //0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
}
```

Method2: channel断言遍历
- 无`close(ch)`: 堵塞，因为channel没有数据+没有close
- 有`close(ch)`: 正常

```go
if value, ok := <-ch; ok {

}
// 如果channel中有数据， value 保存 <-ch 读到的数据。 ok 被设置为 true
// 如果channel中没有数据，channel被close， value 保存 <-ch 读到的0数据。 ok 被设置为 false
// 如果channel中没有数据，channel没被close， <-ch 堵塞
```

```go
// 有close(ch)
package main

import "fmt"

func main() {
	ch := make(chan int, 20)
	N := 10

	go func() {
		for i := 0; i < N; i++ {
			ch <- i
		}
		close(ch)
	}()

	for {
		if value, ok := <-ch; ok {
			fmt.Printf("%v, ", value)
		} else {
			break
		}
	} // 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
}
```

Method3: `for range`遍历
- 无`close(ch)`: 堵塞，因为channel没有数据+没有close
- 有`close(ch)`: 正常

```go
// 有close(ch)
package main

import "fmt"

func main() {
	ch := make(chan int, 20)
	N := 10

	go func() {
		for i := 0; i < N; i++ {
			ch <- i
		}
		close(ch)
	}()

	for v := range ch {
		fmt.Printf("%v, ", v)
	} // 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
}
```

Method1~3都是子routine写，主routine读；如果都是子routine读写
> 尽量加上`close`, 尤其是多个channel存在的情况下，`for range`, `value, ok:=<-ch`没有`close`会导致堵塞

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan int, 20)
	N := 10

	go func() {
		for i := 0; i < N; i++ {
			ch <- i
		}
		close(ch)
	}()

	go func() {
		// 2N > N
		for i := 0; i < 2*N; i++ {
			fmt.Printf("%v, ", <-ch)
		}
	}() // 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 

	time.Sleep(3 * time.Second)
}
```

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan int, 20)
	N := 10

	go func() {
		for i := 0; i < N; i++ {
			ch <- i
		}
		close(ch)
	}()

	go func() {
		for {
			if value, ok := <-ch; ok {
				fmt.Printf("%d, ", value)
			} else {
				break
			}
		}
	}() // 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,

	time.Sleep(3 * time.Second)
}
```

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan int, 20)
	N := 10

	go func() {
		for i := 0; i < N; i++ {
			ch <- i
		}
		close(ch)
	}()

	go func() {
		for v := range ch {
			fmt.Printf("%v, ", v)
		}
	}() // 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,

	time.Sleep(3 * time.Second)
}
```

## others

example: 主协程与次协程并发

```go
package main

import (
	"fmt"
	"time"
)

func test() {
	var i int
	for {
		fmt.Println(i)
		time.Sleep(time.Second)
		i++
	}
}

func main() {
	go test()
	for {
		fmt.Println("running in main")
		time.Sleep(time.Second)
	}
}
```

MPG模型:
> [MPG model](https://i6448038.github.io/2017/12/04/golang-concurrency-principle/)
- M: Machine, 关联内核线程
- P: Processor, 代表M所需的上下文
- G: Goroutine

example: get CPU number

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	num := runtime.NumCPU()
	fmt.Println(num) // 4
	// // version < 1.8
	// runtime.GOMAXPROCS(4)
}
```

example: goroutine communication
> 采用`go build -race main.go`, `./main.exe`来查看是否有数据竞争
- Method1: 全局变量+锁
- Method2: channel

```go
// Method1
package main

import (
	"fmt"
	"sync"
)

// global variable for goroutine communication
var m = make(map[int]int)
var lock sync.Mutex

func factorial(n int) {
	res := 1
	for i := 1; i <= n; i++ {
		res *= i
	}

	lock.Lock()
	m[n] = res
	lock.Unlock()
}

func main() {
	var wg sync.WaitGroup

	for i := 1; i < 10; i++ {
		go factorial(i)
	}

	wg.Wait()
	fmt.Printf("%#v\n", m) //map[int]int{1:1, 2:2, 3:6, 4:24, 5:120, 6:720, 7:5040, 8:40320, 9:362880}
}
```

channel:
- 类似UNIX的pipe
- 先进先出
- 线程安全，多个goroutine访问，不需要加锁
- channel支持任意类型数据

example: 一个goroutine写channel，主goroutine读取channel

```go
package main

import (
	"fmt"
)

type Student struct {
	Name string
	Age  int
}

func addStudents(stuChan chan *Student) {
	for i := 0; i < 12; i++ {
		newStu := &Student{fmt.Sprintf("stu%d", i), 20 + i}
		stuChan <- newStu
	}
	close(stuChan)
}

func main() {
	stuChan := make(chan *Student, 100)
	go addStudents(stuChan)

	// close channel之后，不再向channel写入数据，主协程可以用for range遍历
	for v := range stuChan {
		fmt.Printf("%#v\n", v)
	}
}
```

example: 两个子routine分别读写channel

```go
package main

import (
	"fmt"
	"runtime"
)

type Student struct {
	Name string
	Age  int
}

func addStudents(stuChan chan *Student) {
	for i := 0; i < 12; i++ {
		newStu := &Student{fmt.Sprintf("stu%d", i), 20 + i}
		stuChan <- newStu
	}
	close(stuChan)
}

func readStudents(stuChan chan *Student) {
	for v := range stuChan {
		fmt.Printf("%#v\n", v)
	}
}

func main() {
	stuChan := make(chan *Student, 100)
	go addStudents(stuChan)
	go readStudents(stuChan)

	for {
		runtime.GC()
	}
}
```

```go
package main

import (
	"fmt"
	"time"
)

type Student struct {
	Name string
	Age  int
}

func addStudents(stuChan chan *Student) {
	for i := 0; i < 12; i++ {
		newStu := &Student{fmt.Sprintf("stu%d", i), 20 + i}
		stuChan <- newStu
	}
	close(stuChan)
}

func readStudents(stuChan chan *Student) {
	for {
		if v, ok := <-stuChan; ok {
			fmt.Printf("%#v\n", v)
		} else {
			break
		}
	}
}

func main() {
	stuChan := make(chan *Student, 100)
	go addStudents(stuChan)
	go readStudents(stuChan)

	time.Sleep(3 * time.Second)
}
```

example: 一堆goroutine写channel，主goroutine读取channel

```go
package main

import (
	"fmt"
)

type Student struct {
	Name string
	Age  int
}

func addStudent(i int, stuChan chan *Student) {
	newStu := &Student{fmt.Sprintf("stu%d", i), 20 + i}
	stuChan <- newStu
}

func main() {
	stuChan := make(chan *Student, 100)

	N := 12
	for i := 0; i < N; i++ {
		go addStudent(i, stuChan)
	}

	// N必须相等，否则堵塞(因为无法close channel, 所以不能用for range遍历)
	for i := 0; i < N; i++ {
		fmt.Printf("%#v\n", <-stuChan)
	}
}
```

example: 阻塞现象
> 一个routine瞬间写入10  
> 另一个routine一个一个读，前一个routine一个一个加

```go
package main

import (
	"fmt"
	"time"
)

type Student struct {
	Name string
	Age  int
}

func addStudents(stuChan chan *Student) {
	for i := 0; i < 20; i++ {
		newStu := &Student{fmt.Sprintf("stu%d", i), 20 + i}
		fmt.Printf("Add: %#v\n", newStu)
		stuChan <- newStu
	}
	close(stuChan)
}

func readStudents(stuChan chan *Student) {
	for stu := range stuChan {
		fmt.Printf("%#v\n", stu)
		time.Sleep(time.Second)
	}
}

func main() {
	stuChan := make(chan *Student, 10)
	go addStudents(stuChan)
	go readStudents(stuChan)

	time.Sleep(30 * time.Second)
}
```

example: 两个channel

```go
package main

import (
	"fmt"
)

func sendData(ch chan int, exitChn chan bool) {
	for i := 0; i < 10; i++ {
		ch <- i
	}
	close(ch)
	exitChn <- true
}

func getData(ch chan int, exitChn chan bool) {
	for v := range ch {
		fmt.Printf("%d, ", v)
	}
	exitChn <- true
}

func main() {
	ch := make(chan int, 10)

	N := 2
	// 完成信号
	exitChn := make(chan bool, N)

	go sendData(ch, exitChn)
	go getData(ch, exitChn)

	for i := 0; i < N; i++ {
		// 两个操作都结束才退出
		<-exitChn
	}
}
```

```go
package main

import (
	"fmt"
)

func Modify(i int, taskChn chan int, resultChn chan int) {
	for v := range taskChn {
		// taskChn的尺寸不断变化
		fmt.Printf("routine: %d, len:%d\n", i, len(taskChn))
		v *= 10
		resultChn <- v
	}
}

func main() {
	N := 100
	taskChn := make(chan int, N)
	resultChn := make(chan int, N)

	go func() {
		for i := 0; i < N; i++ {
			taskChn <- i
		}
		close(taskChn)
	}()

	for i := 0; i < 8; i++ {
		go Modify(i, taskChn, resultChn)
	}

	// 因为resultChn无法close, 所以只能用
	for i := 0; i < N; i++ {
		fmt.Printf("%v, ", <-resultChn)
	}
}
```

example: 三个channel

```go
package main

import (
	"fmt"
)

func Modify(i int, taskChn chan int, resultChn chan int, exitChn chan int) {
	for v := range taskChn {
		v *= 10
		resultChn <- v
	}
	exitChn <- i
}

func main() {
	N := 100
	taskChn := make(chan int, N)
	resultChn := make(chan int, N)

	Num := 8
	exitChn := make(chan int, Num) // 退出信号

	go func() {
		for i := 0; i < N; i++ {
			taskChn <- i
		}
		close(taskChn)
	}()

	for i := 0; i < Num; i++ {
		go Modify(i, taskChn, resultChn, exitChn)
	}

	go func() {
		for i := 0; i < Num; i++ {
			fmt.Printf("routine-%v exit!\n", <-exitChn)
		}
		close(resultChn)
	}()

	// 只有resultChn close才能for range
	for v := range resultChn {
		fmt.Printf("%d, ", v)
	}
}
```

example: read-only & write-only channel

```go
package main

import (
	"fmt"
	"time"
)

// write-only channel
func sendData(ch chan<- int) {
	for i := 0; i < 10; i++ {
		ch <- i
	}
	close(ch)
}

// read-only channel
func getData(ch <-chan int) {
	for {
		if v, ok := <-ch; ok {
			fmt.Printf("%v, ", v)
		} else {
			break
		}
	}
}

func main() {
	ch := make(chan int, 10)

	go sendData(ch)
	go getData(ch)

	time.Sleep(time.Second)
}
```

example: channel `select`

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	c1 := make(chan string)
	c2 := make(chan string)

	go func() {
		time.Sleep(1 * time.Second)
		c1 <- "one"
	}()
	go func() {
		time.Sleep(2 * time.Second)
		c2 <- "two"
	}()
forloop:
	for {
		select {
		case msg1 := <-c1:
			fmt.Println("received", msg1)
		case msg2 := <-c2:
			fmt.Println("received", msg2)
			break forloop
		default:
			fmt.Println("not block")
		}
		time.Sleep(500 * time.Millisecond)
	}
}
// not block
// not block
// received one
// not block
// received two
```

example: ticker

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.NewTicker(time.Second) // 每second触发一次
	for v := range t.C {             // t.C本质是channel
		fmt.Printf("%v\n", v.Format("2006/1/02 15:04:05"))
	}
}
```

example: timeout by `time.After`

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan int, 10)
	go func() {
		time.Sleep(3 * time.Second)
		ch1 <- 666
	}()

	select {
	case v1 := <-ch1:
		fmt.Printf("v1=%v\n", v1)
	case <-time.After(time.Second): // 1s后执行，总共执行一次
		fmt.Println("get data timeout")
		break
	}
}
```

example: timeout by `time.NewTicker`

```go
package main

import (
	"fmt"
	"time"
)

func queryDb(ch chan int) {
	time.Sleep(3 * time.Second)
	ch <- 100
}
func main() {
	ch := make(chan int)

	go queryDb(ch)
	t := time.NewTicker(time.Second)
	select {
	case v := <-ch:
		fmt.Println("result", v)
	case <-t.C:
		fmt.Println("timeout")
	}
	// 用完ticker务必关闭, 防止内存泄漏
	t.Stop()
}
```

goroutine with panic: 如果某个goroutine panic了，而且这个goroutine 里面没有捕获(recover)，那么整个进程就会挂掉。所以，好的习惯是每当go产生一个goroutine，就需要写下recover

example: goroutine with panic

```go
package main

import (
	"fmt"
	"time"
)

func calc() {

	defer func() { //defer必须放置在最前面，才能捕获后面所有的panic，程序退出时执行defer
		err := recover() //捕获goroutine错误
		if err != nil {
			fmt.Println(err)
		}
	}()

	var p *int
	*p = 100
}

func otherWork() {
	for {
		fmt.Println("doing other work")
		time.Sleep(time.Second)
	}
}

func main() {
	go calc()
	go otherWork()
	time.Sleep(time.Second * 10)
}
```

## unit test

- 文件名必须以`_test.go`结尾
- 使用`go test` or `go test -v`执行单元测试

example: simple test

```bash
src/
	project1/
		main/
			main.go
			calc.go
			calc_test.go
			student.go
			student_test.go
```

```go
// main.go
package main

func main() {

}
```

```go
// calc.go
package main

func add(a, b int) int {
	return a + b
}

func sub(a, b int) int {
	return a - b
}
```

```go
// calc_test.go
package main

import (
	"testing"
)

func TestAdd(t *testing.T) { //TestAdd必须大写的Test开头
	result := add(2, 8) //测试add函数
	if result != 10 {
		t.Fatalf("add is not right") //致命错误记录
		return
	}

	t.Logf("add is right") //记录一些常规信息
}

func TestSub(t *testing.T) {
	result := sub(2, 8)
	if result != -6 {
		t.Fatalf("sub is not right")
		return
	}

	t.Logf("sub is right")
}
```

```go
// student.go
package main

import (
	"encoding/json"
	"io/ioutil"
)

type Student struct {
	Name string
	Age  int
}

func (s *Student) Save() (err error) {
	data, e := json.Marshal(s)
	if e != nil {
		return
	}

	err = ioutil.WriteFile("stu.txt", data, 0666)
	return
}

func (s *Student) Load() (err error) {
	data, e := ioutil.ReadFile("stu.txt")
	if e != nil {
		return
	}
	err = json.Unmarshal(data, s)
	return
}
```

```go
// student_test.go
package main

import (
	"testing"
	"time"
)

func TestSave(t *testing.T) {
	stu := &Student{"alpha", 66}
	err := stu.Save()
	if err != nil {
		t.Fatalf("save file error")
	}
}

func TestLoad(t *testing.T) {
	stu1 := &Student{"alpha", 66}
	err := stu1.Save()
	if err != nil {
		t.Fatalf("save file error")
	}

	stu2 := &Student{}
	time.Sleep(10 * time.Second) // 中途修改文件，演示测试错误
	err = stu2.Load()
	if err != nil {
		t.Fatalf("load file error")
	}

	if stu1.Name != stu2.Name {
		t.Fatalf("name not match")
	}
	if stu1.Age != stu2.Age {
		t.Fatalf("Age not match")
	}
}
```

```bash
Administrator@PC20180310 MINGW32 ~/Downloads/src/project1/main
$ go test -v
=== RUN   TestAdd
--- PASS: TestAdd (0.00s)
    calc_test.go:14: add is right
=== RUN   TestSub
--- PASS: TestSub (0.00s)
    calc_test.go:24: sub is right
=== RUN   TestSave
--- PASS: TestSave (0.00s)
=== RUN   TestLoad
--- FAIL: TestLoad (10.00s)
    student_test.go:34: Age not match
FAIL
exit status 1
FAIL    _/C_/Users/Administrator/Downloads/src/project1/main    10.024s
```