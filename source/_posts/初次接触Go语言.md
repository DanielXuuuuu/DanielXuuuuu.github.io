---
title: 初次接触Go语言
date: 2019-09-12 17:09:35
categories: 
	- 服务计算
tags: 
	- Go语言	
---

> 在配置完Go语言编程环境后，我阅读了官方文档 [如何使用Go编程](https://go-zh.org/doc/code.html) ，并按文档写第一个包，做第一次测试。
>

{% asset_img go.jpg %}

<!-- more-->

> 项目地址：https://github.com/DanielXuuuuu/ServiceComputingOnCloud

##### 准备工作

首先，需要了解一些基础知识：

###### 编程环境

+ **工作空间**：Go代码必须放在工作空间内。它其实就是一个目录，其中包含三个子目录：

  + `src` 目录包含Go的源文件，它们被组织成包（每个目录都对应一个包）;
  + `pkg` 目录包含编译后生成的包/库文件；
  + `bin` 目录包含编译后生成的可执行文件。
+ **环境变量**：`GOPATH` 环境变量指定了你的工作空间位置（工作空间目录），需要添加到系统的PATH中。`GOPATH` 适合处理大量 Go语言源码、多个包组合而成的复杂工程。

###### GO相关

+ 包 Package

  + Go语言是使用包来组织源代码的，并实现命名空间的管理。任何源代码文件必须属于某个包。源码文件的第一行有效代码必须是 package pacakgeName 语句，通过该语句声明自己所在的包。Go 语言的入口 main() 函数所在的包（package）叫 main，main 包想要引用别的代码，必须同样以包的方式进行引用。
  + 包一般放到公司的域名目录下，这样能保证包名的唯一性，便于共享代码。比如个人的 GitHub 项目的包一般放到 $GOPATH/src/github.com/userName/projectName 目录下。
  + 包的定义是不包括目录路径的，但是包的引用一般是全路径引用

+ [go工具](https://go-zh.org/cmd/go/)：在命令行下，go提供了很多自带的命令行工具：

  ```
  Go is a tool for managing Go source code.
  
  Usage:
  
      go <command> [arguments]
  
  The commands are:
  
      bug         start a bug report
      build       compile packages and dependencies
      clean       remove object files and cached files
      doc         show documentation for package or symbol
      env         print Go environment information
      fix         update packages to use new APIs
      fmt         gofmt (reformat) package sources
      generate    generate Go files by processing source
      get         download and install packages and dependencies
      install     compile and install packages and dependencies
      list        list packages or modules
      mod         module maintenance
      run         compile and run Go program
      test        test packages
      tool        run specified go tool
      version     print Go version
      vet         report likely mistakes in packages
  ```

##### 我的第一个程序

以下是已设置好的环境变量：

{% asset_img gopath.png %}

首先在src目录下创建一个文件夹，并在其中新建一个文件

```/
mkdir $GOPATH/src/github.com/DanielXuuuuu/hello
cd $GOPATH/src/github.com/DanielXuuuuu/hello
touch hello.go
```

通过vim或者VSCode输入以下代码

```
package main

import "fmt"

func main() {
	fmt.Printf("Hello, world.\n")
}
```

保存后，在终端执行` go install github.com/DanielXuuuuu/hello`，go工具会根据GOPATH指定的工作空间，到src子文件夹查找源码。此命令会构建 `hello` 命令，产生一个可执行的二进制文件。 接着它会将该二进制文件作为 `hello`（在 Windows 下则为 `hello.exe`）安装到工作空间的 `bin` 目录中。

由于已经将 `$GOPATH/bin` 添加到 `PATH` 中了，只需输入该二进制文件名即可运行。

{% asset_img hello1.png %}

##### 我的第一个包

首先创建包目录：

```/
$ mkdir $GOPATH/src/github.com/DanielXuuuuu/stringutil
```

接着，在该目录中创建名为 `reverse.go` 的文件：

```
// stringutil 包含有用于处理字符串的工具函数。
package stringutil

// Reverse 将其实参字符串以符文为单位左右反转。
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```

执行`go build github.com/DanielXuuuuu/stringutil`进行编译。编译完后，`stringutil` 包就构建完毕了，我们修改原来的 `hello.go` 文件：

```
package main

import (
	"fmt"

	"stringutil"
)

func main() {
	fmt.Printf(stringutil.Reverse("!oG ,olleH"))
}
```

保存后再次执行`go install ...`，并运行，结果为：

{% asset_img hello2.png %}

##### 编写测试

Go拥有一个轻量级的测试框架，它由 `go test` 命令和 `testing` 包构成。通过创建一个名字为`XXX_test.go` 用于测试`XXX.go`。因此，我们在`stringutil`目录下创建名为 `reverse_test.go` 的文件，代码如下：

```
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

执行`go test github.com/DanielXuuuuu/stringutil`进行测试，结果如下：

{% asset_img test.png %}

输出为`ok`，表明通过测试。