# Go Environment

Golang in Linux:
1. FQ
2. download *.tar.gz from https://golang.org/dl/ and extract
3. setting environment
4. install vscode with go extention

```bash
# setting environment
export GOROOT=$PATH:/path/to/go/
export PATH=$PATH:$GOROOT/bin/
export GOPATH=/home/user/project/go
```

Golang in Ubuntu:
1. `sudo apt install go`
2. install vscdoe with go extension

Golang in windows:
1. FQ
2. download *.zip from https://golang.org/dl/
3. install vscode with go extension
4. go extension setting

```json
{
    // vscode setting for golang
    "go.goroot": "D:\\ProgrammingTools\\go",
    "go.inferGopath": true,
    "go.toolsGopath": "D:\\ProgrammingTools\\gotools",
}
```

example: test environment

```go
package main

import "fmt"

func main() {
	fmt.Println("hello, world!")
}
```