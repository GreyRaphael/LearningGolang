# Go Environment

Best Method to install gotools
- `go env -w GO111MODULE=on`
- `go env -w GOPROXY=https://goproxy.cn,direct`
- install vscode with go extention
- vscode notify gotools installation "Install All"

Golang in Linux:
1. FQ
2. download *.tar.gz from https://golang.org/dl/ and extract
3. setting environment
4. install vscode with go extention
5. vscode notify gotools installation "Install All"

```bash
# setting environment
export GOROOT=$PATH:/path/to/go/
export PATH=$PATH:$GOROOT/bin/
export GOPATH=/home/user/project/go
```

Golang in Ubuntu:
1. `sudo apt install go`
2. install vscdoe with go extension
3. vscode notify gotools installation "Install All"

Golang in windows:
1. FQ
2. download *.zip from https://golang.org/dl/
3. install vscode with go extension
4. go extension setting
5. vscode notify gotools installation "Install All"

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

- Run Code in terminal: `go run helloworld.go`
- Run Code in VSCode: `Ctrl+F5`
- Debug code: `F5`