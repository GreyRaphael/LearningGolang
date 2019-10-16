# Golang IO

- [Golang IO](#golang-io)
  - [File](#file)
  - [json](#json)
  - [Error](#error)

## File

`os.File`封装所有文件相关操作

操作终端相关文件句柄常量，它们都是都是`*os.File`
- `os.Stdin`
- `os.Stdout`
- `os.Stderr`

```go
const (
	O_RDONLY int = syscall.O_RDONLY // 只读打开文件和os.Open()同义
	O_WRONLY int = syscall.O_WRONLY // 只写打开文件
	O_RDWR   int = syscall.O_RDWR   // 读写方式打开文件
	O_APPEND int = syscall.O_APPEND // 当写的时候使用追加模式到文件末尾
	O_CREATE int = syscall.O_CREAT  // 如果文件不存在，此案创建
	O_EXCL   int = syscall.O_EXCL   // 和O_CREATE一起使用, 只有当文件不存在时才创建
	O_SYNC   int = syscall.O_SYNC   // 以同步I/O方式打开文件，直接写入硬盘.
	O_TRUNC  int = syscall.O_TRUNC  // 如果可以的话，当打开文件时先清空文件
)
```

文件权限控制:
- `4`: r
- `2`: w
- `1`: x

example: write to terminal & file

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// print to terminal
	fmt.Fprintln(os.Stdout, "hello")

	// print to file
	file, err := os.OpenFile("data.txt", os.O_CREATE|os.O_WRONLY, 0666)
	if err != nil {
		fmt.Println("open file error!")
		return
	}
	fmt.Fprintf(file, "this is data.txt")
	file.Close()
}
```

example: `Sscanf`

```go
package main

import (
	"fmt"
)

var (
	firstName, lastName, s string
	i                      int
	f                      float32
	input                  = "56.12 / 5212 / Go"
	format                 = "%f / %d / %s"
)

func main() {
	fmt.Println("Please enter your full name: ")
	// fmt.Scanf("%s %s", &firstName, &lastName)
	fmt.Scanln(&firstName, &lastName)              // 只能读string, 且以空格分隔
	fmt.Printf("Hi %s %s!\n", firstName, lastName) // Hi Chris Naegels

	fmt.Sscanf(input, format, &f, &i, &s)
	fmt.Println("From the string we read: ", f, i, s)
}
```

example `Sscanf` with struct

```go
package main

import (
	"fmt"
)

type Student struct {
	Name  string
	Age   int
	Score float32
}

func main() {
	str1 := "stu01 24 56.5"
	stu1 := Student{}
	fmt.Sscanf(str1, "%s %d %f", &stu1.Name, &stu1.Age, &stu1.Score)
	fmt.Printf("%#v\n", stu1) //main.Student{Name:"stu01", Age:24, Score:56.5}
}
```

example: `bufio` Reader
> 带缓冲区的读写

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

type Student struct {
	Name  string
	Age   int
	Score float32
}

func main() {
	inputReader := bufio.NewReader(os.Stdin)
	fmt.Println("enter you name:")
	input, err := inputReader.ReadString('\n')
	if err == nil {
		fmt.Println("your name:", input)
	}
}
```

example: `bufio` write

```go
package main

import (
	"bufio"
	"os"
)

func main() {
	fi, _ := os.OpenFile("test.txt", os.O_WRONLY|os.O_CREATE, 0666)
	defer fi.Close()

	writer := bufio.NewWriter(fi)
	for i := 0; i < 10; i++ {
		writer.WriteString("hello, world\n")
	}
	writer.Flush() // bufio必须有
}
```

example: bufio write to terminal

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	fmt.Fprintf(os.Stdout, "%s\n", "hello world! - unbuffered")
	buf := bufio.NewWriter(os.Stdout)
	fmt.Fprintf(buf, "%s\n", "hello world! - buffered")
	buf.Flush()
}
```

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

type Student struct {
	Name  string
	Age   int
	Score float32
}

func main() {
	file, err := os.Open("data.txt")
	if err != nil {
		return
	}
	defer file.Close()

	inputReader := bufio.NewReader(file)
	str, err := inputReader.ReadString('\n') // read 1 line
	if err != nil {
		return
	}
	fmt.Println(str)
}
```

> 文件需要`Close()`, 如果在Linux只允许存在65535的文件句柄存在(通过`ulimit -a`查看))

example: 统计文件中英文、空格、数字、其他字符

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
	"strings"
)

type CharCounter struct {
	Ch    int
	En    int
	Space int
	Num   int
	Other int
}

func (counter *CharCounter) Count(lines []string) {
	for _, line := range lines {
		for _, c := range line {
			if c >= 'a' && c <= 'z' || c >= 'A' && c <= 'Z' {
				counter.En++
			} else if c >= '0' && c <= '9' {
				counter.Num++
			} else if c == ' ' {
				counter.Space++
			} else if c >= 19968 && c <= 40869 {
				// Unicode汉字编码范围\u4E00-\u9FA5: 19968~40869
				counter.Ch++
			} else {
				counter.Other++
			}
		}
	}
	return
}

func main() {
	// open file
	file, err := os.Open("data.txt")
	if err != nil {
		return
	}
	defer file.Close()

	// bufio reader
	inputReader := bufio.NewReader(file)
	var lines []string
	for {
		str, err := inputReader.ReadString('\n') // read 1 line
		str = strings.TrimRight(str, "\r\n")     // for windows
		if err == io.EOF {
			// 补充最后一行
			lines = append(lines, str)
			break
		}
		if err != nil {
			fmt.Println("read file error", err)
			return
		}
		lines = append(lines, str)
	}
	counter := CharCounter{}
	counter.Count(lines)
	fmt.Printf("%#v\n", counter) // main.CharCounter{Ch:7, En:13, Space:4, Num:6, Other:4}
}
```

example: copy file by `ioutil`

```go
package main

import (
	"fmt"
	"io/ioutil"
)

func main() {
	buf, _ := ioutil.ReadFile("data.txt")
	fmt.Printf("%v", buf) // type is []byte

	err := ioutil.WriteFile("data_copy.txt", buf, 0x666)
	if err != nil {
		panic(err.Error())
	}
}
```

exampl: copy file by `io.Copy()`
- 从文件到文件
- 从文件到显示器
- 从文件到网络

```go
package main

import (
	"fmt"
	"io"
	"os"
)

func main() {

	CopyFile("target.txt", "source.txt")
	fmt.Println("Copy done!")
}

func CopyFile(dstName, srcName string) (written int64, err error) {
	src, err := os.Open(srcName)
	if err != nil {
		return
	}
	defer src.Close()

	dst, err := os.OpenFile(dstName, os.O_WRONLY|os.O_CREATE, 0644)
	if err != nil {
		return
	}
	defer dst.Close()

	return io.Copy(dst, src)
}
```

example: read `.gz`

```go
package main

import (
	"bufio"
	"compress/gzip"
	"fmt"
	"io"
	"os"
)

func main() {
	fi, _ := os.Open("file1.gz") // gz内部只能有一个文件，多个文件需要先打包成tar
	fz, _ := gzip.NewReader(fi)
	r := bufio.NewReader(fz)
	for {
		line, err := r.ReadString('\n')
		fmt.Printf(line)
		if err == io.EOF {
			fmt.Println("\ndone")
			os.Exit(0)
		}
	}
}
```

example: `os.Args`是一个string的切片，用来存储所有的命令行参数

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	for _, v := range os.Args {
		fmt.Println(v)
	}
}
```

example: `flag`用来解析命令行参数

```go
package main

import (
	"flag"
	"fmt"
)

func main() {
	var configPath string
	var logLevel int
	flag.StringVar(&configPath, "c", "", "please input config path")
	flag.IntVar(&logLevel, "d", 3, "please input log level")
	flag.Parse()
	fmt.Println(configPath, logLevel)
}
// in terminal > ./main.exe -c "c:/config" -d 6
// c:/config 6
```

example: read multi files

```go
package main

import (
	"bufio"
	"flag"
	"fmt"
	"io"
	"os"
)

func cat(r *bufio.Reader) {
	for {
		buf, err := r.ReadBytes('\n')
		fmt.Fprintf(os.Stdout, "%s", buf)
		if err == io.EOF {
			break
		}
	}
}

func main() {
	flag.Parse()
	fmt.Println(flag.Args())

	// command line没有参数从terminal读取
	if flag.NArg() == 0 {
		cat(bufio.NewReader(os.Stdin))
	}

	// command line有参数从文件读取
	for i := 0; i < flag.NArg(); i++ {
		f, err := os.Open(flag.Arg(i))
		if err != nil {
			fmt.Fprintf(os.Stderr, "%s:error reading from %s: %s\n", os.Args[0], flag.Arg(i), err.Error())
			continue
		}
		cat(bufio.NewReader(f))
	}
}
```

## json

exmaple: 序列化`struct`, `map`

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Student struct {
	Name  string
	Age   int `json:"stu_age"`
	Score float32
}

func main() {
	stu1 := &Student{"alpha", 23, 12.5}

	// struct to json
	data1, _ := json.Marshal(stu1)
	fmt.Printf("%#v\n", data1)         // []byte
	fmt.Printf("%#v\n", string(data1)) // "{\"Name\":\"alpha\",\"stu_age\":23,\"Score\":12.5}"

	// map to json
	m1 := map[string]interface{}{"Name": "beta", "stu_age": 36, "Score": 13.6}
	data2, _ := json.Marshal(m1)
	fmt.Printf("%#v\n", string(data2)) // "{\"Name\":\"beta\",\"Score\":13.6,\"stu_age\":36}"
}
```

example: 反序列化`struct`, `map`

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Student struct {
	Name  string
	Age   int `json:"stu_age"`
	Score float32
}

func main() {
	str := `{"Name":"alpha","Score":12.5,"stu_age":32}`

	// json to sturct
	stu1 := &Student{}
	json.Unmarshal([]byte(str), stu1)
	fmt.Printf("%#v\n", stu1) //&main.Student{Name:"alpha", Age:32, Score:12.5}

	// json to map
	m1 := map[string]interface{}{}
	json.Unmarshal([]byte(str), &m1)
	fmt.Printf("%#v\n", m1) // map[string]interface {}{"Name":"alpha", "Score":12.5, "stu_age":32}

	m2 := make(map[string]interface{})
	json.Unmarshal([]byte(str), &m2)
	fmt.Printf("%#v\n", m2) // map[string]interface {}{"Name":"alpha", "Score":12.5, "stu_age":32}

	var m3 map[string]interface{}
	json.Unmarshal([]byte(str), &m3)
	fmt.Printf("%#v\n", m3) // map[string]interface {}{"Name":"alpha", "Score":12.5, "stu_age":32}
}
```

example: 序列化`slice`

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Student struct {
	Name  string
	Age   int `json:"stu_age"`
	Score float32
}

func main() {
	stu1 := Student{"alpha", 23, 12.5}
	stu2 := Student{"beta", 34, 12.5}
	var students []Student
	students = append(students, stu1, stu2)

	data, _ := json.Marshal(students)
	fmt.Printf("%#v\n", string(data)) // "[{\"Name\":\"alpha\",\"stu_age\":23,\"Score\":12.5},{\"Name\":\"beta\",\"stu_age\":34,\"Score\":12.5}]"
}
```

examle: 反序列化`slice`

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Student struct {
	Name  string
	Age   int `json:"stu_age"`
	Score float32
}

func main() {
	str := `[{"Name":"alpha","stu_age":23,"Score":12.5},{"Name":"beta","stu_age":34,"Score":12.5}]`

	// json to slice of struct
	var students []Student
	json.Unmarshal([]byte(str), &students)
	fmt.Printf("%#v\n", students) // []main.Student{main.Student{Name:"alpha", Age:23, Score:12.5}, main.Student{Name:"beta", Age:34, Score:12.5}}

	// json to slice of map
	var stus []map[string]interface{}
	json.Unmarshal([]byte(str), &stus)
	fmt.Printf("%v\n", stus) // [map[Name:alpha Score:12.5 stu_age:23] map[Name:beta Score:12.5 stu_age:34]]
}
```

## Error

golang中的`error`是如下接口

```go
type error interface {
    Error() string
}
```

example: 自定义错误实现error的接口

```go
package main

import (
	"fmt"
	"os"
	"time"
)

type PathError struct {
	Op         string
	Path       string
	Msg        string
	CreateTime string
}

func (e *PathError) Error() string {
	return fmt.Sprintf("%v|%v|%v|%v\n", e.Op, e.Path, e.Msg, e.CreateTime)
}

func Test(filename string) error {
	file, err := os.Open(filename)
	if err != nil {
		return &PathError{"read", filename, err.Error(), time.Now().Format("2006/1/02")}
	}
	defer file.Close()
	return nil
}

func main() {
	err := Test("xxx.txt")
	if err != nil {
		fmt.Println(err.Error())
	}

	// type assertion method1
	v, ok := err.(*PathError)
	if ok {
		fmt.Printf("this is custom PathError:%#v\n", v)
		// this is custom PathError:&main.PathError{Op:"read", Path:"xxx.txt", Msg:"open xxx.txt: The system cannot find the file specified.", CreateTime:"2019/10/16"}
	}

	// type assertion method2
	switch v := err.(type) {
	case *PathError:
		fmt.Printf("this is custom PathError:%#v\n", v)
	default:

	}
}
```

example: 一般错误都不采用上述方法

```go
package main

import (
	"errors"
	"fmt"
	"os"
)

var PathError = errors.New("PathError")

func Test(filename string) error {
	file, err := os.Open(filename)
	if err != nil {
		return PathError
	}
	defer file.Close()
	return nil
}

func main() {
	err := Test("xxx.txt")
	if err != nil {
		fmt.Println(err)         // PathError
		fmt.Println(err.Error()) // PathError
	}
}
```

example: `panic` & `recover`

```go
package main

import (
	"fmt"
)

func badCall() {
	panic("bad end")
}

func test() {
	defer func() {
		if e := recover(); e != nil {
			fmt.Printf("Panicking %s\n", e)
		}
	}()
	badCall()
	fmt.Printf("After bad call\r\n")
}

func main() {
	fmt.Printf("Calling test\r\n")
	test()
	fmt.Printf("Test completed\r\n")
}
```