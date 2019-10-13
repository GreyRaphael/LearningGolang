# Golang Interface

interface类型可以定义一组方法，但是这些方法不需要实现。并且interface不能包含任何变量。
> 遵循一个interface就是遵循一个规范，令使用方不用关注具体的实现

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
	var p Phone

	p = Xiaomi{}
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

	phone = NokiaPhone{}
	phone.call()

	phone = IPhone{}
	phone.call()
}
```

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

func (nokiaPhone *NokiaPhone) call() {
	fmt.Println("I am Nokia, I can call you!")
}

type IPhone struct {
}

func (iPhone *IPhone) call() {
	fmt.Println("I am iPhone, I can call you!")
}

func main() {
	var phone Phone

	phone = &NokiaPhone{}
	phone.call()
	fmt.Printf("%#v\n", phone) //&main.NokiaPhone{}

	phone = &IPhone{}
	phone.call()
	fmt.Printf("%#v\n", phone) //&main.IPhone{}
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
- Golang中的接口，不需要显式地实现(比如Java中的关键字`implement`)。只要一个变量，含有接口类型中的**所有方法**，那么这个变量就实现这个接口。
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
	playSugar()
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

// 两个方法都实现了才算实现了Android接口
func (p Xiaomi) playSugar() {
	fmt.Println("I'm playing sugar game")
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

example: 接口嵌套

```go
package main

import "fmt"

type Reader interface {
	Read()
}

type Writer interface {
	Write()
}

type ReadWriter interface {
	// 接口嵌套
	Reader
	Writer
}

type File struct {
}

func (f *File) Read() {
	fmt.Println("file read")
}

func (f *File) Write() {
	fmt.Println("file write")
}

func Test(rw ReadWriter) {
	rw.Read()
	rw.Write()
}

func main() {
	var f *File
	Test(f)
}
```

空接口: 空接口没有任何方法，所以所有类型都实现了空接口，也就是任何变量都可以赋值给空接口

```go
package main

import "fmt"

func main() {
	var a interface{}
	fmt.Printf("%v, %T\n", a, a) // <nil>, <nil>
	var b int
	b = 10
	a = b
	fmt.Printf("%v, %T\n", a, a) // 10, int
}
```

example: `interface{}`切片

```go
package main

import "fmt"

func main() {
	var a []interface{}
	a = append(a, 10)
	a = append(a, "grey")
	a = append(a, 12.56)
	for _, v := range a {
		fmt.Printf("%v, ", v)
	} // 10, grey, 12.56,
}
```

example: 普通struct实现Sort接口
> [Sort interface](https://go-zh.org/pkg/sort/#Sort)

```go
// 要实现该Interface这个类型
type Interface interface {
    // Len is the number of elements in the collection.
    // Len 为集合内元素的总数
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    //
    // Less 返回索引为 i 的元素是否应排在索引为 j 的元素之前。
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    // Swap 交换索引为 i 和 j 的元素
    Swap(i, j int)
}
```

```go
package main

import (
	"fmt"
	"math/rand"
	"sort"
	"time"
)

type Student struct {
	Name string
	Age  int
}

// 通过vscode的codesnippet生成
type StuSlice []Student

func (a StuSlice) Len() int           { return len(a) }
func (a StuSlice) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a StuSlice) Less(i, j int) bool { return a[i].Age < a[j].Age }

func init() {
	rand.Seed(time.Now().UnixNano())
}

func main() {
	var students StuSlice
	for i := 0; i < 5; i++ {
		newStu := Student{Name: fmt.Sprintf("stu%d", i), Age: rand.Intn(20)}
		students = append(students, newStu)
	}
	fmt.Println(students)
	sort.Sort(students)
	fmt.Println(students)
}
// [{stu0 6} {stu1 0} {stu2 12} {stu3 12} {stu4 18}]
// [{stu1 0} {stu0 6} {stu2 12} {stu3 12} {stu4 18}]
```

类型断言: 从普通类型到接口变量直接赋值就行；从接口变量得到传入的类型，就需要类型断言。

example: simple type assertion

```go
package main

import "fmt"

func main() {
	var a interface{}
	var b int

	a = b // 普通类型→interface, 直接赋值
	c := a.(int) // interface→普通类型
	fmt.Printf("%T, %v\n", c, c) //int, 0
}
```

example: type assertion function

```go
package main

import "fmt"

func Test(i interface{}) {
	res, ok := i.(int)
	if ok {
		fmt.Printf("Result=%v\n", res)
	} else {
		fmt.Println("not int")
	}
}

func main() {
	a := 100
	b := "grey"
	Test(a) // Result=100
	Test(b) // not int
}
```

example: classifier

```go
package main

import "fmt"

type Student struct{}

func classifier(items ...interface{}) {
	for i, v := range items {
		switch v.(type) {
		case bool:
			fmt.Printf("param #%d is a bool\n", i)
		case float64:
			fmt.Printf("param #%d is a float64\n", i)
		case int:
			fmt.Printf("param #%d is an int\n", i)
		case nil:
			fmt.Printf("param #%d is nil\n", i)
		case string:
			fmt.Printf("param #%d is a string\n", i)
		case *Student:
			fmt.Printf("param #%d is a *Student\n", i)
		default:
			fmt.Printf("param #%d is type is unknown\n", i)
		}
	}
}

func main() {
	a := 10
	b := "grey"
	c := &Student{}
	classifier(a, b, c)
}
// param #0 is an int
// param #1 is a string
// param #2 is a *Student
```

example: 判断一个变量是否实现了指定接口

```go
package main

import "fmt"

type Person interface {
	Eat()
}

type Student struct{}

func (s Student) Eat() {
	fmt.Println("I'm eating")
}

func main() {
	s := Student{}
	var i interface{} = s
	v, ok := i.(Person)
	fmt.Println(v, ok) // {} true
}
```

example: 实现一个通用的链表

```go
package main

import "fmt"

type LinkNode struct {
	data interface{}
	next *LinkNode
}

type Link struct {
	head *LinkNode
	tail *LinkNode
}

func (l *Link) LeftAppend(data interface{}) {
	node := &LinkNode{data: data}
	// 空链表判断
	if l.head == nil && l.tail == nil {
		l.head = node
		l.tail = node
	} else {
		node.next = l.head
		l.head = node
	}
}

func (l *Link) RightAppend(data interface{}) {
	node := &LinkNode{data: data}
	// 空链表判断
	if l.head == nil && l.tail == nil {
		l.head = node
		l.tail = node
	} else {
		l.tail.next = node
		l.tail = node
	}
}

func (l *Link) Traverse() {
	head := l.head
	for head != nil {
		fmt.Printf("%v, ", head.data)
		head = head.next
	}
	fmt.Println()
}

func main() {
	var l Link
	for i := 0; i < 3; i++ {
		l.LeftAppend(fmt.Sprintf("boy-%d", i))
	}
	for i := 0; i < 3; i++ {
		l.RightAppend(fmt.Sprintf("girl-%d", i))
	}
	l.Traverse() // boy-2, boy-1, boy-0, girl-0, girl-1, girl-2, 
}
```