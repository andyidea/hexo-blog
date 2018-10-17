---
title: Go Modules使用教程
date: 2018-10-17 11:15:47
tags: [golang]
---

## 引入

[https://talks.godoc.org/github.com/myitcv/talks/2018-08-15-glug-modules/main.slide#1](https://talks.godoc.org/github.com/myitcv/talks/2018-08-15-glug-modules/main.slide#1)

## Go Modules介绍

Modules是Go 1.11中新增的实验性功能，基于vgo演变而来，是一个新型的包管理工具。


## 常见的包管理工具

- govendor
- dep
- glide
- godep

这些包管理工具都是基于`GOPATH`或者`vendor`目录，并不能很好的解决不同版本依赖问题。Modules是在`GOPATH`之外一套新的包管理方式。



## 如何激活Modules

首先要把go升级到1.11。

升级后，可以设置通过一个环境变量`GO111MODULE`来激活modules：

- GO111MODULE=off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。
- GO111MODULE=on，go命令行会使用modules，而一点也不会去GOPATH目录下查找。
- GO111MODULE=auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。这种情况下可以分为两种情形：当前目录在GOPATH/src之外且该目录包含go.mod文件，或者当前文件在包含go.mod文件的目录下面。


当module功能启用时，`GOPATH`在项目构建过程中不再担当import的角色，但它仍然存储下载的依赖包，具体位置在`$GOPATH/pkg/mod`。



## 初始化Modules

Go1.11新增了命令`go mod`来支持Modules的使用。

```
> go help mod
Go mod provides access to operations on modules.

Note that support for modules is built into all the go commands,
not just 'go mod'. For example, day-to-day adding, removing, upgrading,
and downgrading of dependencies should be done using 'go get'.
See 'go help modules' for an overview of module functionality.

Usage:

	go mod <command> [arguments]

The commands are:

	download    download modules to local cache
	edit        edit go.mod from tools or scripts
	graph       print module requirement graph
	init        initialize new module in current directory
	tidy        add missing and remove unused modules
	vendor      make vendored copy of dependencies
	verify      verify dependencies have expected content
	why         explain why packages or modules are needed

Use "go help mod <command>" for more information about a command.
```

首先创建一个项目helloworld：

```
cd && mkdir helloworld && cd helloworld
```

然后创建文件`main.go`并写入：

```
package main

import (
    log "github.com/sirupsen/logrus"
)

func main() {
    log.WithFields(log.Fields{
        "animal": "walrus",
    }).Info("A walrus appears")
}
```

初始化mod:

```
go mod init helloworld
```

系统生成了一个`go.mod`的文件：

```
module helloworld

```

然后执行go build，再次查看go.mod文件发现多了一些内容：

```
module helloworld

require github.com/sirupsen/logrus v1.1.1

```

同时多了一个`go.sum`的文件：

```
github.com/davecgh/go-spew v1.1.1/go.mod h1:J7Y8YcW2NihsgmVo/mv3lAwl/skON4iLHjSsI+c5H38=
github.com/konsorten/go-windows-terminal-sequences v0.0.0-20180402223658-b729f2633dfe/go.mod h1:T0+1ngSBFLxvqU3pZ+m/2kptfBszLMUkC4ZK/EgS/cQ=
github.com/pmezard/go-difflib v1.0.0/go.mod h1:iKH77koFhYxTK1pcRnkKkqfTogsbg7gZNVY4sRDYZ/4=
github.com/sirupsen/logrus v1.1.1 h1:VzGj7lhU7KEB9e9gMpAV/v5XT2NVSvLJhJLCWbnkgXg=
github.com/sirupsen/logrus v1.1.1/go.mod h1:zrgwTnHtNr00buQ1vSptGe8m1f/BbgsPukg8qsT7A+A=
github.com/stretchr/testify v1.2.2/go.mod h1:a8OnRcib4nhh0OaRAV+Yts87kKdq0PP7pXfy6kDkUVs=
golang.org/x/crypto v0.0.0-20180904163835-0709b304e793 h1:u+LnwYTOOW7Ukr/fppxEb1Nwz0AtPflrblfvUudpo+I=
golang.org/x/crypto v0.0.0-20180904163835-0709b304e793/go.mod h1:6SG95UA2DQfeDnfUPMdvaQW0Q7yPrPDi9nlGo2tz2b4=
golang.org/x/sys v0.0.0-20180905080454-ebe1bf3edb33 h1:I6FyU15t786LL7oL/hn43zqTuEGr4PN7F4XJ1p4E3Y8=
golang.org/x/sys v0.0.0-20180905080454-ebe1bf3edb33/go.mod h1:STP8DvDyc/dI5b8T5hshtkjS+E42TnysNCUPdjciGhY=

```
go.sum不是一个锁文件，是一个模块版本内容的校验值，用来验证当前缓存的模块。go.sum包含了直接依赖和间接依赖的包的信息，比go.mod要多一些。


    
## go.mod

有四种指令：module，require，exclude，replace。

- module：模块名称
- require：依赖包列表以及版本
- exclude：禁止依赖包列表（仅在当前模块为主模块时生效）
- replace：替换依赖包列表 （仅在当前模块为主模块时生效）


## 其他命令


```
go mod tidy //拉取缺少的模块，移除不用的模块。
```

```
go mod download //下载依赖包
```

```
go mod graph //打印模块依赖图
```

```
go mod vendor //将依赖复制到vendor下
```

```
go mod verify //校验依赖
```

```
go mod why //解释为什么需要依赖
```

```
go list -m -json all //依赖详情
```
