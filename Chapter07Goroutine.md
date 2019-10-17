# Golang goroutine

- [Golang goroutine](#golang-goroutine)
	- [channel traverse](#channel-traverse)
	- [others](#others)

Process vs Thread vs Coroutine:
- 进程是程序在操作系统中的一次执行过程，系统进行资源分配和调度的一个独立单位
- 线程是进程的一个执行实体，是CPU调度和分派的基本单位，是比进程更小的能独立运行的基本单位
- 一个进程可以创建和销毁多个线程；同一个进程中的多个线程之间可以并发执行
- 协程：独立的栈空间，共享堆空间，调度由用户控制，类似用户级线程；一个线程上可以跑多个协程

## channel traverse

Method1: channel简单遍历(not recommended)
- 读次数<=写次数: 正常
- 读次数>写次数: 
  - 无`close(ch)`: deadlock
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
	} //deadlock
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
- 无`close(ch)`: deadlock
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
- 无`close(ch)`: deadlock
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
		// close(ch) // 不带close更好，带了之后会多出0
	}()

	go func() {
		for i := 0; i < 2*N; i++ {
			fmt.Printf("%v, ", <-ch)
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
	"sync"
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
	var wg sync.WaitGroup

	stuChan := make(chan *Student, 100)
	go addStudents(stuChan)

	wg.Wait()

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
}

func readStudents(stuChan chan *Student) {
	// // method1:
	// for v := range stuChan {
	// 	fmt.Printf("%#v\n", v)
	// }

	// method2:
	for {
		fmt.Printf("%#v\n", <-stuChan)
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
	"sync"
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
	var wg sync.WaitGroup

	stuChan := make(chan *Student, 100)

	N := 12
	for i := 0; i < N; i++ {
		go addStudent(i, stuChan)
	}

	wg.Wait()
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
}

func readStudents(stuChan chan *Student) {
	for {
		fmt.Printf("%#v\n", <-stuChan)
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
	"time"
)

func Modify(i int, taskChn chan int, resultChn chan int) {
	for v := range taskChn {
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
	}()

	for i := 0; i < 8; i++ {
		go Modify(i, taskChn, resultChn)
	}

	go func() {
		for v := range resultChn {
			fmt.Printf("%d, ", v)
		}
	}()

	time.Sleep(5 * time.Second)
}
```

