---
title: "Go"
date: 2021-01-21
tags: ["Go"]
---

# Go

## Install

~/.local 以下にインストールしていくケースです  

```
cd ~/.local
curl -OL https://go.dev/dl/gox.y.z.linux-amd64.tar.gz
tar zxf gox.y.z.linux-amd64.tar.gz
mkdir -p gopath/bin
```

.bashrc に以下を追記します	

```
export PATH=~/.local/go/bin:~/.local/gopath/bin:$PATH
export GOPATH=~/.local/gopath
```

## Update

```
cd ~/.local
rm -r go
curl -OL https://go.dev/dl/gox.y.z.linux-amd64.tar.gz
tar zxf gox.y.z.linux-amd64.tar.gz
```

## hello, world

```
$ mkdir hello
$ cd hello
$ go mod init hello
```

hello.go をつくります  
インデントはタブ 1つ です  

```
package main

import "fmt"

func main() {
	fmt.Println("hello, world")
}
```

```
$ go run hello.go
hello, world
```

## go mod

[Using Go Modules](https://blog.golang.org/using-go-modules)  
[go mod init <ここには何を書く？>](https://teratail.com/questions/217859)  

```
$ # go mod init [公開しなければこちらはなんでもいいらしい]
$
$ # 例
$ go mod init hello
```

## go test

```
go mod init mul
vi mul_test.go	# The file name must end with _test.go
```

```
package main

import "testing"

func mul(x int, y int) int {
	return x * y
}

func TestMul(t *testing.T) {  // The function name must begin with Test and the camel case follows
	if mul(3, 19) != 57 {
		t.Errorf("Failed")
	}
}
```

## vi

スペースをタブに変換する  

```
:set noet
:ret!
```

タブをスペースに変換する  

```
:set et
:ret
```
