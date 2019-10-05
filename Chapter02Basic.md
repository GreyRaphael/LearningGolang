# Golang Basic

- 任何一个代码文件隶属于一个包
- 同一个package中的函数直接调用，不同package中的函数通过`package1.func1()`调用
- 小写字母开头的函数/变量对于一个package来说是private; 大写字母开头函数/变量是public

`_`: 特殊标识符，用来忽略结果

keywords:
- import, package, 
- switch, select, case, if, else, default, fallthrough
- for, continue, break, range, goto
- func, return, defer, var, type, const, struct, map
- interface 
- go, chan

example: for loop

```go
package main

import "fmt"

func main() {
	for i := 0; i < 6; i++ {
		fmt.Println(i, "+", 6-i, "=", 6)
	}
}
```

example: package private & public

```bash
src/
    project1/
        calc/
            add.go
        main/
            main.go
```

situation1

```go
// add.go
package calc

// 必须要这么写，不能将声明与初始化分开
var Name string = "Grey"
var age int = 36
```

```go
// main.go
package main

import (
	"fmt"
	"project1/calc"
)

func main() {
	fmt.Println(calc.Name)
	// fmt.Println(calc.age) // cannot access
}
```

situation2

```go
// add.go
package calc

var Name string
var age int

func Test() {
	Name = "Grey"
	age = 36
}
```

```go
// main.go
package main

import (
	"fmt"
	"project1/calc"
)

func main() {
	calc.Test()
	fmt.Println(calc.Name)
}
```

example: package alias

```go
// main.go
package main

import (
	"fmt"
	mycalc "project1/calc"
)

func main() {
	mycalc.Test()
	fmt.Println(mycalc.Name)
}
```

example: `init` function for single file
> 每一个package中的执行顺序(注意嵌套关系):
> 1. 调用`import`函数
> 1. 初始化全局变量， 如果有的话
> 1. 调用`init`函数，如果有的话
> 1. 调用`main`函数，如果有的话

```go
package main

import "fmt"

func main() {
	for i := 0; i < 6; i++ {
		fmt.Println(i, "+", 6-i, "=", 6)
	}
}

func init() {
	fmt.Println("Before main")
}
```

example: `init` function for files

```bash
src/
    project1/
        calc/
            add.go
        main/
            main.go
```

```go
// add.go
package calc

var Name string
var age int

func init() {
	Name = "Grey"
	age = 36
}
```

```go
// main.go
package main

import (
	"fmt"
	mycalc "project1/calc"
)

func main() {
	fmt.Println(mycalc.Name)
}
```

example: `import` 表示执行该package内容

```bash
src/
    project1/
        calc/
            add.go
        main/
            main.go
```

```go
// add.go
package calc

import "fmt"

var Name string
var age int

func init() {
	Name = "Grey"
	age = 36
	fmt.Println("this is package: calc")
}
```

```go
// main.go
package main

import (
	"fmt"
	_ "project1/calc" // calc仍然会被执行
)

func main() {
	fmt.Println("This is package: main")
}
```

example: const
> const 只能修饰boolean，number（int相关类型、浮点类型、complex）和string。  
> `const indentifier [type] = value`, `type`可以省略

```go
const b string = "Grey"
const c = "Alpha"
const Pi = 3.1415926
const a = 9/3
```

example: const trick, `iota`

```go
package main

import "fmt"

func main() {
	const (
		a = 0
		b = "Grey"
		c = 3.1415926
	)

	const (
		d = iota
		e
		f
	)
	fmt.Println(d, e, f) // 0 1 2
}
```

example: if statement

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	const (
		Male   = 1
		Female = 2
	)
	for i := 0; i < 10; i++ {
		second := time.Now().Unix()
		if second%2 == 0 {
			fmt.Println("Female")
		} else {
			fmt.Println("Male")
		}
		time.Sleep(time.Second)
	}
}
```

example: variable

```go
package main

import "fmt"

func main() {
	var (
		a int     // 默认0
		b bool    // 默认false
		c string  //默认 ""
		d float32 = 3.14
		f         = "james"
	)
	a = 10
	b = true
	c = "grey"
	fmt.Println(a, b, c, d, f)

	var a0 int
	var b0 bool
	var c0 string
	var d0 float64
	a0 = 11
	b0 = true
	c0 = "alpha"
	d0 = 1.414
	fmt.Println(a0, b0, c0, d0)

	var a1 = 111
	var b1 = false
	var c1 = "mori"
	var d1 = 2.718
	fmt.Println(a1, b1, c1, d1)
}
```

example: get `PATH`

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	var goos = os.Getenv("GOOS")
	fmt.Printf("os=%s\n", goos)
	path := os.Getenv("PATH")
	fmt.Printf("PATH=%s\n", path)
}
```

值类型vs引用类型
> 无论是值传递，还是引用传递，传递给函数的都是变量的副本，不过，值传递是值的拷贝。引用传递是地址的拷贝，一般来说，地址拷贝更为高效。而值拷贝取决于拷贝的对象大小，对象越大，则性能越低。
- 值类型：变量直接存储值，内存通常在栈中分配。
  - 基本数据类型int、float、bool、string以及数组和struct。
- 引用类型：变量存储的是一个地址，这个地址存储最终的值。内存通常在堆上分配。通过GC回收。
  - 指针、slice、map、chan等都是引用类型。

example: value & ref

```go
package main

import (
	"fmt"
)

func main() {
	a := 10
	b := make(chan int, 3)
	fmt.Println(a, b) // 10 0xc00001a080
}
```

```go
package main

import (
	"fmt"
)

func modifyA(a int) {
	a = 100
	return
}

func modifyB(p *int) {
	*p = 100
	return
}

func main() {
	a := 10
	modifyA(a)
	fmt.Println(a) //10
	modifyB(&a)
	fmt.Println(a) //100
}
```

example: swap two number

```go
package main

import "fmt"

func swap(p1 *int, p2 *int) {
	temp := *p1
	*p1 = *p2
	*p2 = temp
}

func easySwap(a int, b int) (int, int) {
	return b, a
}

func main() {
	a := 10
	b := 5
	swap(&a, &b)
	fmt.Println(a, b)
	a, b = b, a
	fmt.Println(a, b)
	a, b = easySwap(a, b)
	fmt.Println(a, b)
}
```

作用域：
- 语句体中声明的变量，生命周期仅限于语句体
- 在函数内部声明的变量叫做局部变量，生命周期仅限于函数内部。
- 在函数外部声明的变量叫做全局变量，生命周期作用于整个包，如果是大写的，则作用于整个程序。

example: 注意局部变量与全局变量

```go
package main

import "fmt"

var a = 10

func n() {
	fmt.Println(a)
}

func m() {
	// a是全局变量
	a = 5
	fmt.Println(a)
}

func main() {
	n() //10
	m() //5
	n() //5
}
```

```go
package main

import "fmt"

var a = 10

func n() {
	fmt.Println(a)
}

func m() {
	// a是局部变量
	a := 5
	fmt.Println(a)
}

func main() {
	n() //10
	m() //5
	n() //10
}
```

example: attention

```go
package main

import "fmt"

var a int=10

// 因为全局变量在函数外部，语句不会被执行，只能如上面初始化
// 下面两种做法等价
// 错误做法1
var a int
a=10

//错误做法2
a:=10
```

```go
package main

import "fmt"

func main() {
	var a int
	var b int32
	a = 100
	// int, int32虽然范围一样，但是仍然是不同类型
	// b = a + a // compile err
	b = int32(a) + int32(a)
	fmt.Println(b)
}
```

example: `math/rand`

```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	for i := 0; i < 10; i++ {
		fmt.Println(rand.Int31(), rand.Int31n(10), rand.Float32())
	}
}
// 最终结果是一样的，需要进一步随机
```

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func init() {
	rand.Seed(time.Now().UnixNano())
}

func main() {
	for i := 0; i < 10; i++ {
		fmt.Println(rand.Int31(), rand.Int31n(10), rand.Float32())
	}
}
```

负载均衡策略:
- LVS: Nginx
- RPC

example: byte vs string

```go
package main

import (
	"fmt"
)

func main() {
	var c byte
	c = 'a'
	var str string
	str = "Grey\n"

	// raw string
	var str2 string
	str2 = `Alpha\n
	Beta
	Gamma`
	fmt.Println(c, str, str2)
	// 97 Grey
	// Alpha\n
	//		  Beta
    //		  Gamma
    fmt.Printf("%c", c) // a
}
```

example: number to string

```go
package main

import (
	"fmt"
)

func main() {
	// num2string
	str := fmt.Sprintf("%d%%", 10)
	fmt.Println(str) // 10%
}
```

example: string operation

```go
package main

import "fmt"

func main() {
	// method1
	str1 := "Alpha"
	str2 := "Grey"
	str3 := str1 + str2
	fmt.Println(str3) // AlphaGrey
	// method2
	str4 := fmt.Sprintf("%s%s", str1, str2)
	fmt.Println(str4, len(str4)) //AlphaGrey 9

	// substring
	fmt.Println(str4[5:]) // Grey
}
```

example: string reverse

```go
package main

import "fmt"

func strReverse1(s string) string {
	runes := []rune(s)
	for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
		runes[i], runes[j] = runes[j], runes[i]
	}
	return string(runes)
}

func strReverse2(s string) string {
	var result string
	strLen := len(s)
	for i := 0; i < strLen; i++ {
		result += fmt.Sprintf("%c", s[strLen-i-1])
	}
	return result
}

func strReverse3(s string) string {
	var result []byte
	tmp := []byte(s)
	strLen := len(s)
	for i := 0; i < strLen; i++ {
		result = append(result, tmp[strLen-i-1])
	}
	return string(result)
}

func main() {
	str1 := "helloworld"
	fmt.Println(strReverse1(str1))
	fmt.Println(strReverse2(str1))
	fmt.Println(strReverse3(str1))
}
```

输入网址发生的过程:
- 输入网址，解析出IP：如果浏览器有缓存，直接获取IP;没有缓存访问DNS服务器;
- 根据IP经历TCP三次握手建立连接，发送数据包
- 一般远端的Nginx接受到包，解析协议，拿到URL，然后Nginx Proxy转发给后台的服务器
- 如果有缓存，直接返回缓存；没有缓存，访问redis, 数据库来渲染HTML，HTML给Nginx Proxy
- Nginx Proxy返回浏览器
- 浏览器渲染成页面

负载均衡方式[load balence](https://www.jishuwen.com/d/2tL2):
- 代理: proxy
- RPC
  - 客户端决定访问服务器，导致笨重的客户端
  - 先访问Lookaside服务器，Lookaside服务器返回负载均衡策略，然后客户端根据策略访问对应服务

exampe: prime number

```go
package main

import (
	"fmt"
	"math"
)

func isPrime(a int) bool {
	for i := 2; i <= int(math.Sqrt(float64(a))); i++ {
		if a%i == 0 {
			return false
		}
	}
	return a > 1
}

func main() {
	count := 0
	for i := 101; i < 200; i += 2 {
		if isPrime(i) {
			count++
			fmt.Println(i)
		}
	}
	fmt.Println("count=", count)
}
```

example: interactive prime

```go
package main

import (
	"fmt"
	"math"
)

func isPrime(a int) bool {
	for i := 2; i <= int(math.Sqrt(float64(a))); i++ {
		if a%i == 0 {
			return false
		}
	}
	return a > 1
}

func main() {
	var m int
	var n int
	fmt.Scanf("%d%d", &m, &n) // 输入时空格来分割

	count := 0
	for i := m; i < n; i += 2 {
		if isPrime(i) {
			count++
			fmt.Println(i)
		}
	}
	fmt.Println("count=", count)
}
```

example: 水仙花

```go
package main

import "fmt"

func isShui(a int) bool {
	i := a % 10
	j := a / 10 % 10
	k := a / 100
	return a == i*i*i+j*j*j+k*k*k
}

func main() {
	count := 0
	for i := 100; i < 1000; i++ {
		if isShui(i) {
			count++
			fmt.Println(i)
		}
	}
	fmt.Println("count=", count)
}
```

example: 水仙花 by string

```go
package main

import (
	"fmt"
)

func isShui(s string) bool {
	var resultA int
	var resultB int

	for i := 0; i < len(s); i++ {
		num := int(s[i] - '0')
		resultA += num * num * num
		resultB = resultB*10 + num
	}
	return resultA == resultB

}

func main() {
	var str string
	fmt.Scanf("%s", &str)
	fmt.Println(isShui(str))
}
```

example: 水仙花 by strconv

```go
package main

import (
	"fmt"
	"strconv"
)

func isShui(s string) bool {
	var resultA int

	for i := 0; i < len(s); i++ {
		num := int(s[i] - '0')
		resultA += num * num * num
	}
	resultB, err := strconv.Atoi(s)
	if err != nil { // err是对象指针，所以用nil
		fmt.Println("convert string error")
	}
	return resultA == resultB
}

func main() {
	var str string
	fmt.Scanf("%s", &str)
	fmt.Println(isShui(str))
}
```

example: sum factorial

```go
package main

import "fmt"

func factorial(n int) int {
	if n == 1 {
		return 1
	}
	return n * factorial(n-1)
}

func main() {
	N := 5
	sum := 0
	for i := 1; i < N; i++ {
		sum += factorial(i)
	}
	fmt.Println(sum)
}
```

example: sumfactorial func

```go
package main

import (
	"fmt"
)

func sumFactorial(n int) int {
	if n == 1 {
		return 1
	}
	f := 1
	for i := 1; i <= n; i++ {
		f = f * i
	}
	return sumFactorial(n-1) + f
}

func main() {
	fmt.Println(sumFactorial(4))
}
```

example: sum factorial loop

```go
package main

import (
	"fmt"
)

func sumFactorial(n int) int {
	f := 1
	sum := 0
	for i := 1; i <= n; i++ {
		f = f * i
		sum += f
	}
	return sum
}

func main() {
	fmt.Println(sumFactorial(4))
}
```

string functions:
- `strings.HasPrefix(str, "http://")`
- `strings.HasSuffix(str, "/")`
- `strings.Index(str, "tp")`
- `strings.LastIndex(str, "dex")`
- `strings.Replace(str string, old string, new string, n int)`
- `strings.Count(str string, substr string)int`
- `strings.Repeat(str string, count int)string`
- `strings.ToLower(str string)string`
- `strings.ToUpper(str string)string`
- `strings.TrimSpace(str string)`
- `strings.Trim(str string, cut string)`, `strings.TrimLeft(str string, cut string)`, `strings.TrimRight(str string, cut string)`
- `strings.Field(str string)`:返回str空格分隔的所有子串的slice
- `strings.Split(str string, split string)`
-`strings.Join(s1 []string, sep string)`
- `strings.Itoa(i int)`: int to string
- `strings.Atoi(str string)(int, error)`: string to int

example: trim

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "abbacba"
	fmt.Println(strings.Trim(str, "ab")) // a
}
```

time package:
- `time.Time`类型表示时间
- `time.Now()`
- `time.Duration`表示纳秒

```go
now = time.Now()
fmt.Printf("%02d/%02d%02d %02d:%02d:%02d", now.Year(), now.Month(), now.Day(), now.Hour(), now.Minute(), now.Second())

// some const
const(
	Nanosecond Duration =1
	Microsecond = 1000*Nanosecond
	Millisecond = 1000*Microsecond
	Second      = 1000*Millisecond
	Minute		= 60*Second
	Hour		= 60*Minute
)
```

```go
// time format
now := time.Now()
fmt.Println(now.Format(“02/1/2006 15:04”))
fmt.Println(now.Format(“2006/1/02 15:04”))
fmt.Println(now.Format(“2006/1/02”))
```

example: count time

```go
package main

import (
	"fmt"
	"time"
)

func test() {
	time.Sleep(3 * time.Second)
}

func main() {
	start := time.Now().UnixNano()
	test()
	end := time.Now().UnixNano()
	fmt.Println(end - start) // 3000847500
}
```

流程控制:
- 顺序结构
- 判断结构
- 循环结构

判断结构:

```go
// if-else
if condition1 {

}

if condition1 {

} else{

}

if condition1 {

} else if condition2 {

} else {

}
```

```go
// switch-case, 不需要break
switch value {
	case 0:
	case 1:
	default:
}

// fallthrough
switch value {
	case 0:
		fmt.PrintLn("hello")
		fallthrough // 往下走一层
	case 1:
	case 2:
	default:
}

switch value{
	case 0, 1:
	case 2:
	default:
}

switch {
	condition1:
	condition2:
	condition3:
	default:
}

switch i:=xxx {
	condition1:
	condition2:
	condition3:
	default
}
```

example: `fallthrough`

```go
package main

import (
	"fmt"
)

func main() {
	a := 0
	switch a {
	case 0:
		fallthrough // 往下走一层，可以尝试有无fallthrough
	case 1:
		fmt.Println("level1")
		fallthrough
	case 2:
		fmt.Println("level2")
	case 3:
		fmt.Println("level3")
	default:
		fmt.Println("default")
	}
}
```

example: guess number

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	rand.Seed(time.Now().Unix())
	N := rand.Intn(10)
	var a int
loop:
	for {
		fmt.Scanf("%d\n", &a) // \n是为了回车符号导致打印两次
		switch {
		case a == N:
			fmt.Println("bingo")
			break loop // break loop from swith-case, 或者使用flag
		case a > N:
			fmt.Println("too big")
		case a < N:
			fmt.Println("too small")
		}
	}
}
```

循环结构 for:

```go
for i := 0; i < N; i++ {
	
}

for condition {
	// 类似while循环
}

for true {

}

// 等价于
for {
	// infinity loop
}

//enumerate
for i, v:= range xxx{

}
```

example: while loop

```go
package main

import "fmt"

func main() {
	i := 0
	for i < 3 {
		fmt.Println(i)
		i++
	}
}
```

```go
package main

import "fmt"

func main() {
	i := 0
	for {
		if i >= 3 {
			break
		}

		fmt.Println(i)
		i++
	}
}
```

example: print star

```go
package main

import "fmt"

func main() {
	for i := 0; i < 5; i++ {
		for j := 0; j <= i; j++ {
			fmt.Printf("*")
		}
		fmt.Println()
	}
}
// *
// **
// ***
// ****
// *****
```

example: iterate array
> 还可以遍历slice, map, chan, string

```go
package main

import "fmt"

func main() {
	a := []int{1, 2, 3, 4, 5}
	for i, v := range a {
		fmt.Printf("index=%d, value=%d\n", i, v)
	}
}
```

example: enumerate string utf8

```go
package main

import "fmt"

func main() {
	str := "hi, 中国"
	for i, v := range str {
		fmt.Printf("index:%d,val=%c, len=%d\n", i, v, len([]byte(string(v))))
	}
}
// index:0,val=h, len=1
// index:1,val=i, len=1
// index:2,val=,, len=1
// index:3,val= , len=1
// index:4,val=中, len=3
// index:7,val=国, len=3
```

`goto` & label

example: goto loop

```go
package main

import "fmt"

func main() {
label1:
	for i := 0; i < 4; i++ {
		// label2:
		for j := 0; j < 4; j++ {
			if j == 2 {
				continue label1
			}
			fmt.Println(i, j)
		}
	}
}
// 0 0
// 0 1
// 1 0
// 1 1
// 2 0
// 2 1
// 3 0
// 3 1
```

example: goto loop

```go
package main

func main() {
	i := 0
label1:
	println(i)
	i++
	if i == 5 {
		return
	}
	goto label1
}
```

函数形式

```go
func functionName() {

}

func functionName(a int, b int) {

}

func functionName(a int, b int) int {

}

func functionName(a int, b int) (int, int) {

}

func functionName(a, b int) (int, int) {

}
```

golang函数特点:
- 不支持重载，一个包不能有两个名字一样的函数
- 函数是一等公民，函数也是一种类型，一个函数可以赋值给变量
- 匿名函数
- 多返回值

example: 函数作为变量

```go
package main

import "fmt"

func add(a int, b int) int {
	return a + b
}

func main() {
	c := add
	fmt.Println(c, c(10, 20)) //0x4b7410 30
}
```

example: 函数类型调用

```go
package main

import "fmt"

// 定义一种类型，只是为了简写方便
type two2one func(int, int) int

// func operator(op func(int, int) int, a int, b int) int {
func operator(op two2one, a int, b int) int {
	return op(a, b)
}

func add(a, b int) int {
	return a + b
}

func main() {
	c := add
	fmt.Println(operator(c, 100, 200)) // 300
}
```

值传递vs引用传递
> map、slice、chan、指针、interface默认以引用的方式传递

example: 给返回值命名

```go
package main

import "fmt"

func sumAvg(a, b int) (sum, avg int) {
	sum = a + b
	avg = sum / 2
	return
}

func main() {
	sum, _ := sumAvg(10, 20)
	fmt.Println(sum) // 30
}
```

example: 可变参数

```go
package main

import "fmt"

// >=1个参数
func multiSum(a int, arg ...int) int {
	sum := a
	// arg是一个slice
	// 也可以 arg[index]来访问
	for _, v := range arg {
		sum += v
	}
	return sum
}

func main() {
	fmt.Println(multiSum(10))
	fmt.Println(multiSum(10, 20))
	fmt.Println(multiSum(10, 20, 30))
}
```

```go
package main

import "fmt"

// >=0个参数
func multiSum(arg ...int) int {
	sum := 0
	for _, v := range arg {
		sum += v
	}
	return sum
}

func main() {
	fmt.Println(multiSum())
	fmt.Println(multiSum(10))
	fmt.Println(multiSum(10, 20))
}
```

example: string concatenation

```go
package main

import "fmt"

func concat(s1, s2 string, arg ...string) string {
	sum := s1 + s2
	for _, v := range arg {
		sum += v
	}
	return sum
}

func main() {
	fmt.Println(concat("hello", "world"))
	fmt.Println(concat("hello", "world", "China"))
}
```

