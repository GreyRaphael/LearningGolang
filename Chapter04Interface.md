# Golang Interface

- [Golang Interface](#golang-interface)

Interface: Interface类型可以定义一组方法，但是这些不需要实现。并且interface不能包含任何变量。

```go
type example interface{
    Method1(参数列表) 返回值列表
    Method2(参数列表) 返回值列表
}
```

example: interface implement

```go
package main

import (
	"fmt"
)

type Phone interface {
	send(string) bool
	surf()
}

type Xiaomi struct {
}

// 对interface的实现
func (p Xiaomi) send(data string) bool {
	fmt.Printf("I'm sending %v\n", data)
	return data != ""
}

func (p Xiaomi) surf() {
	fmt.Println("I'm surfing the Internet")
}

func main() {
	p := Xiaomi{}
	p.surf()
	status := p.send(`"hello"`)
	fmt.Println(status) // true
}
```

example: 如果一个变量实现了String()这个方法，那么fmt.Println默认会调用这个变量的String()进行输出。

```go
package main

import "fmt"

type Student struct {
	Name string
	Age  int
}

func (s Student) String() string {
	return fmt.Sprintf("Name=%v,Age=%v\n", s.Name, s.Age)
}

func main() {
	stu1 := Student{"alpha", 15}
	fmt.Printf("%#v\n", stu1) // main.Student{Name:"alpha", Age:15}
	fmt.Println(stu1)         // Name=alpha,Age=15
}
```

example: 多态
> 一种事物的多种形态，都可以按照统一的接口进行操作

```go
package main

import (
	"fmt"
)

type Phone interface {
	call()
}

type NokiaPhone struct {
}

func (nokiaPhone NokiaPhone) call() {
	fmt.Println("I am Nokia, I can call you!")
}

type IPhone struct {
}

func (iPhone IPhone) call() {
	fmt.Println("I am iPhone, I can call you!")
}

func main() {
	var phone Phone

	phone = new(NokiaPhone)
	phone.call()

	phone = new(IPhone)
	phone.call()
}
```

example: interface本质

```go
package main

import (
	"fmt"
)

type Phone interface {
	send(string) bool
	surf()
}

type Xiaomi struct {
}

// 对interface的实现
func (p Xiaomi) send(data string) bool {
	fmt.Printf("I'm sending %v\n", data)
	return data != ""
}

func (p Xiaomi) surf() {
	fmt.Println("I'm surfing the Internet")
}

func main() {
	var p Phone
	// 类型和数值都是<nil>
	fmt.Printf("%#v, %T\n", p, p) // <nil>, <nil>

	p = Xiaomi{}
	fmt.Printf("%#v, %T\n", p, p) //main.Xiaomi{}, main.Xiaomi
}
```

Attension:
- Golang中的接口，不需要显示的实现。只要一个变量，含有接口类型中的**所有方法**，那么这个变量就实现这个接口。因此，golang中没有implement类似的关键字
- 如果一个变量含有了多个interface类型的方法，那么这个变量就实现了多个接口。
- 如果一个变量只含有了1个interface的方部分方法，那么这个变量没有实现这个接口。

example: 一个struct实现多个interface

```go
package main

import (
	"fmt"
)

type Android interface {
	surf()
}

type Phone interface {
	surf()
	send(string) bool
}

type Xiaomi struct {
}

// 实现了Android和Phone两个接口
func (p Xiaomi) surf() {
	fmt.Println("I'm surfing the Internet")
}

// 两个方法都实现了才算实现了Phone接口
func (p Xiaomi) send(data string) bool {
	fmt.Printf("I'm sending %v\n", data)
	return data != ""
}

func main() {
	var a Android
	var p Phone
	mi := Xiaomi{}

	a = mi
	p = mi

	a.surf()
	p.surf()
}
```

interface嵌套

```go
type ReadWrite interface {
	Read(b Buffer) bool
	Write(b Buffer) bool
}
type Lock interface {
	Lock()
	Unlock()
}
type File interface {
	ReadWrite
	Lock
	Close()
}
```

空接口: 空接口没有任何方法，所以所有类型都实现了空接口。

```go
var a int
var b interface{}
b  = a
```


