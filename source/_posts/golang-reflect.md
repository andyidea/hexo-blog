---
title: golang中的reflect包用法
tags: [golang]
---

>最近在写一个自动生成api文档的功能，用到了reflect包来给结构体赋值，给空数组新增一个元素，这样只要定义一个input结构体和一个output的结构体，并填写一些相关tag信息，就能使用程序来生成输入和输出的相关文档。

### 介绍

reflect包是golang中很重要的一个包,实现了在运行时允许程序操纵任意类型对象的功能。可以看下[文档](https://golang.org/pkg/reflect/)简单了解一下。

在reflect中,最重要的是Value类,只有先获取到一个对象或者变量的Value对象后,我们才可以对这个对象或者变量进行更进一步的分析和处理。我们可以使用[reflect.ValueOf()](https://golang.org/pkg/reflect/#ValueOf)方法获取Value对象。  

```go

var i int
value := reflect.ValueOf(i) // 使用ValueOf()获取到变量的Value对象

type S struct {
    a string
}

var s S
value2 := reflect.ValueOf(s) // 使用ValueOf()获取到结构体的Value对象
```
获取到对象或者变量的Value对象后，我们就可以对他们进一步的操作了。

### 1.获取对象或者变量的类型(Value.Type()和Value.Kind())

[Value.Type()](https://golang.org/pkg/reflect/#Value.Type)和[Value.Kind()](https://golang.org/pkg/reflect/#Value.Kind)这两个方法都可以获取对象或者变量的类型，如果是变量的话，使用这两个方法获取到的类型都是一样，差别是结构体对象，举个例子看一下：

```go
var i int
value := reflect.ValueOf(i)

log.Println(value.Type()) //输出:int
log.Println(value.Kind()) //输出:int

type S struct {
    a string
}

var s S
value2 := reflect.ValueOf(s) // 使用ValueOf()获取到结构体的Value对象


log.Println(value2.Type()) //输出:S
log.Println(value2.Kind()) //输出:struct
```

变量`i`使用kind和type两个方法都输出了`int`,而结构体`s`的Type()方法输出了`S`,Kind()方法输出了`struct`，由此可以总结如下，如果你想拿到结构体里面定义的变量信息的时候，使用Type(f)方法。如果只是相判断是否是结构体时，就使用Kind()

### 2.获取变量的值和给变量赋值

获取变量的值使用[value.Interface()](https://golang.org/pkg/reflect/#Value.Interface)方法，该方法会返回一个value的值，不过类型是interface。给变量赋值需要先判断该变量的类型，使用之前提到过的Value.Kind()方法，如果变量的类型是reflect.Int，我们就可以使用[Value.SetInt()](https://golang.org/pkg/reflect/#Value.SetInt)方法给变量赋值。下面是一个例子：

```go
var i int = 1

// 获取Value,这里注意,如果你要改变这个变量的话,需要传递变量的地址
value := reflect.ValueOf(&i)

// value是一个指针,这里获取了该指针指向的值,相当于value.Elem()
value = reflect.Indirect(value)

// Interface是获取该value的值,返回的是一个interface对象
log.Println(value.Interface()) // 输出:1

// 把变量i的值设为2
if value.Kind() == reflect.Int {
	value.SetInt(2)
}

log.Println(value.Interface()) // 输出:2
```
给结构体对象中的成员变量赋值的方法：

```go
type S struct {
	A string // 注意:只有大写开头的成员变量可以Set
}

s := S{"x"}

value := reflect.ValueOf(&s)

value = reflect.Indirect(value)


//value是结构体s,所以打印出来的是整个结构体的信息
log.Println(value.Interface()) //输出: {x}

f0 := value.FieldByName("A") //获取结构体s中第一个元素a

log.Println(f0) // 输出: x

if f0.Kind() == reflect.String {
	if f0.CanSet() {
		f0.SetString("y")
	}
}

log.Println(f0) // 输出: y

log.Println(value.Interface()) //输出: {y}

```

结构体这里需要注意的是，只有公有的成员变量可以被reflect改变值，私有的变量是无法改变值得。

### 3.获取结构体成员变量的tag信息

由于golang变量大小写和公有私有息息相关，所以码农门很难按照自己的意愿来定义变量名。于是golang提供了tag机制，来给变量提供一个标签，这个标签可以作为一个别名，来给一些存储结构来获取结构体变量名字使用。下面是一个获取结构体成员变量tag信息的例子：

```go
type S struct {
	A string `json:"tag_a"`
}

s := S{}

value := reflect.ValueOf(&s)

value = reflect.Indirect(value)

//获取结构体s的类型S
vt := value.Type()

//获取S中的A成员变量
f, _ := vt.FieldByName("A")

//获取成员变量A的db标签
log.Println(f.Tag.Get("json")) //输出: tag_a
```

未完待续。。。