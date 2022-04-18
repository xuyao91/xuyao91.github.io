---
layout: post
title: "Golang项目中的版本管理"
date: 2022-04-18 16:59:05 +0800
comments: true
categories: 
---

go项目中经常会碰到线上编译好的二进制文件不清楚是哪次commit后编译的，每次编译出来的都是一个二进制文件，不利于管理，搞了个go项目的版本管理。
利用go build的 -ldflags 参数，使用帮助命令查看详细信息
<!-- more -->
```
go build --ldflags="--help"
```
参数很多，但我们感兴趣的是 -X
```
usage: link [options] main.o
  -X definition
      add string value definition of the form importpath.name=value
```
-X 参数，指定 importpath.name=value，用于修改变量值。其中 importpath 表示包导入路径，name 是程序中的变量名，value 代表我们想要设定的变量值。
看一下最简单的例子

```
package main

import (
 "fmt"
)

var (
 version = "0.0.0"
)

func main() {
 fmt.Println("version: ", version)
}
```
如果正常编译执行程序，将得到以下结果
```
go build -o main && ./main
version:  0.0.0
```
此时，我们指定 --ldflags  的 -X 参数重新编译执行

```
go build -o main --ldflags="-X 'main.version=1.0.0'" && ./main
version:  1.0.0
```
##### 项目中使用
但是在实际项目中使用，肯定不会这么简单，需要一些更详细的信息，比如版本号，最后一次commit id，当前的branch,编译时间，编译环境等，
集成了一个包，以后可以在项目中直接用，github地址 [version](https://github.com/xuyao91/version)
原理跟上面写的一样，作用--ldflags="-X"写实现变量值设定，example下面有个shell脚本，执行./build.sh脚本，会直接build项目并设定版本信息，最后运行
```
./example -version
```
输出结果
```
Version: v1.0
Go Version: go version go1.15.6 darwin/amd64
Git Commit: 40a0e5935dcf1286c0c4e4b787167b04211e0324
Git Branch: master
Build Time: 2022-04-12 13:04:47
```
这样就很方便管理编译后的二进制包了。
