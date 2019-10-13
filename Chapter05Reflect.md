# Golang Reflect

Reflect可以在运行时动态获取变量的相关信息
- `reflect.TypeOf`: get type
- `reflect.ValueOf`: get value
- `v.Kind`: 
- `v.Interface()`: convert to `interface{}`

转换关系:
变量←→`interface{}`←→`reflect.Value`

```go
package main

import (
	"fmt"
	"reflect"
)

type Student struct {
	Name string
	Age  int
}

func Test(i interface{}) {
	t := reflect.TypeOf(i)
	v := reflect.ValueOf(i)
	k := v.Kind() //代表类型的一个uint
	fmt.Printf("%#v\n", t)
	fmt.Printf("%#v\n", v)
	fmt.Printf("%#v\n", k)

	//
	iv := v.Interface()
	stu, ok := iv.(Student)
	if ok {
		fmt.Printf("%#v\n", stu)
	}
}

func main() {
	Test(100)
	fmt.Println()
	Test("grey")
	fmt.Println()
	Test(Student{})
}
```

```bash
&reflect.rtype{size:0x8, ptrdata:0x0, hash:0xf75371fa, tflag:0x7, align:0x8, fieldAlign:0x8, kind:0x2, alg:(*reflect.typeAlg)(0x5956f0), gcdata:(*uint8)(0x501f56), str:1095, ptrToThis:51232}
100
0x2

&reflect.rtype{size:0x10, ptrdata:0x8, hash:0xe0ff5cb4, tflag:0x7, align:0x8, fieldAlign:0x8, kind:0x18, alg:(*reflect.typeAlg)(0x595710), gcdata:(*uint8)(0x501f56), str:6832, ptrToThis:62432}
"grey"
0x18

&reflect.rtype{size:0x18, ptrdata:0x8, hash:0xd4209fda, tflag:0x7, align:0x8, fieldAlign:0x8, kind:0x19, alg:(*reflect.typeAlg)(0x4be620), gcdata:(*uint8)(0x501f56), str:26320, ptrToThis:52896}
main.Student{Name:"", Age:0}
0x19
main.Student{Name:"", Age:0}
```

example: reflect simple

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {

	var x float64 = 3.4
	fmt.Println("type:", reflect.TypeOf(x)) // type: float64
	v := reflect.ValueOf(x)
	fmt.Println("value:", v)         // value:3.4
	fmt.Println("type:", v.Type())   // type: float64
	fmt.Println("kind:", v.Kind())   // kind: float64
	fmt.Println("value:", v.Float()) // value:3.4

	fmt.Println(v.Interface())                    // 3.4
	y := v.Interface().(float64)
	fmt.Println(y) //3.4
}
```

example: 获取变量的值
- `v.Float()`
- `v.String()`
- `v.Int()`

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {

	var x float64 = 3.4
	v := reflect.ValueOf(x)
	fmt.Printf("%v\n", v.Float())
	fmt.Printf("%v\n", v.String()) // <float64 Value>
	fmt.Printf("%v\n", v.Bool())   // panic
	fmt.Printf("%v\n", v.Int())    // panic
}
```

example: 修改变量的值
- `v.Elem().SetFloat()`
- `v.Elem().SetString()`
- `v.Elem().SetInt()`

> `func (v Value) SetInt(x int64)`: SetInt sets v's underlying value to x

```go
package main

import (
	"fmt"
	"reflect"
)

func test(i interface{}) {
	fmt.Printf("%T, %v\n", i, i) // *int, 0xc000012098

	val := reflect.ValueOf(i) // val是i的一个拷贝，所以修改的时候要用指针; 如果不是指针而Set会触发panic
	fmt.Printf("%T, %v\n", val, val) // reflect.Value, 0xc000012098

	ve := val.Elem()
	fmt.Printf("%T, %v\n", ve, ve) // reflect.Value, 100

	ve.SetInt(666)
}

func main() {
	a := 100
	test(&a)
	fmt.Println(a) // 666
}
```

example: reflect adv

```go
package main

import (
	"fmt"
	"reflect"
)

type Phd struct {
	Student
}

type Student struct {
	Name string
	Age  int
}

func (s Student) SayHello() {
	fmt.Println("Hello")
}

func test(i interface{}) {
	val := reflect.ValueOf(i)
	fmt.Printf("%T, %#v\n", val, val) // reflect.Value, main.Student{Name:"", Age:0}

	kd := val.Kind()
	if kd != reflect.Struct {
		fmt.Println("expect struct")
		return
	}
	fmt.Printf("Field Num=%d\n", val.NumField())   //2
	fmt.Printf("Method Num=%d\n", val.NumMethod()) //1

	for i := 0; i < val.NumField(); i++ {
		fmt.Printf("Field%d: %v, %v\n", i, val.Field(i), val.Field(i).Kind())
	}

	var params []reflect.Value // nil
	val.Method(0).Call(params) // Hello
}

func main() {
	s := Student{}
	fmt.Printf("%T, %#v\n", s, s) // main.Student, main.Student{Name:"", Age:0}
	p := Phd{}
	fmt.Printf("%T, %#v\n", p, p) // main.Phd, main.Phd{Student:main.Student{Name:"", Age:0}}
	test(s)
}
```

```go
package main

import (
	"fmt"
	"reflect"
)

type NotknownType struct {
	s1 string
	s2 string
	s3 string
}

func (n NotknownType) Concat() string {
	return n.s1 + "-" + n.s2 + "-" + n.s3
}

var secret interface{} = NotknownType{"Ada", "Go", "Oberon"}

func main() {
	value := reflect.ValueOf(secret)
	typ := reflect.TypeOf(secret)
	fmt.Println(typ) // main.NotknownType

	knd := value.Kind() // struct
	fmt.Println(knd)

	for i := 0; i < value.NumField(); i++ {
		fmt.Printf("Field %d: %v\n", i, value.Field(i))
	}

	results := value.Method(0).Call(nil)
	fmt.Printf("%T, %v", results, results) // []reflect.Value, [Ada-Go-Oberon]
}
```

example: modify struct by reflect

```go
package main

import (
	"fmt"
	"reflect"
)

type Student struct {
	Name string
	Age  int
}

func test(a interface{}) {
	ve := reflect.ValueOf(a).Elem()
	ve.Field(0).SetString("gogogo")
	ve.Field(1).SetInt(66)
}

func main() {
	s := &Student{}
	test(s)
	fmt.Println(s) // &{gogogo 66}
}
```

```go
package main

import (
	"fmt"
	"reflect"
)

type T struct {
	A int
	B string
}

func main() {
	t := T{23, "skidoo"}
	s := reflect.ValueOf(&t).Elem()
	typeOfT := s.Type()

	for i := 0; i < s.NumField(); i++ {
		f := s.Field(i)
		fmt.Printf("%d: %s %s = %v\n", i, typeOfT.Field(i).Name, f.Type(), f.Interface())
	}
	s.Field(0).SetInt(77)
	s.Field(1).SetString("Sunset Strip")
	fmt.Println("t=", t) // t= {77 Sunset Strip}
}
```

example: reflect with tag

```go
package main

import (
	"fmt"
	"reflect"
)

type Student struct {
	Name string `json:"student_name"`
	Age  int    `json:"student_age"`
}

func test(a interface{}) {
	ty := reflect.TypeOf(a).Elem()
	f1 := ty.Field(0).Tag.Get("json")
	f2 := ty.Field(1).Tag.Get("json")
	fmt.Println(f1, f2) // student_name student_age
}

func main() {
	s := &Student{}
	test(s)
	fmt.Println(s)
}
```
