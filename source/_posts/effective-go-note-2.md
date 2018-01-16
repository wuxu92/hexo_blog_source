---
date: 2015/07/19 12:34:50
title: Effective Go笔记：方法，接口与其他类型
categories:
- programming
tags:
- golang
- lang
---

effective go 中的部分笔记，这是第二部分 

部分笔记摘要,参考： [https://go-zh.org/doc/effective_go.html](https://go-zh.org/doc/effective_go.html "https://go-zh.org/doc/effective_go.html")

## 方法/Methods ##
与一般的面向对象语言不同；一般的面向对象语言定义类，然后在类中定义属性和方法，通过类的继承来抽象一套机制，但是在Go中，首先是定义结构，然后为已经命名的结构（除了指针或接口）定义方法，这里有一个方法接收者的概念，为一个结构体绑定方法的常用用法如下：

```
type Book struct {
	price float64
}

func (p Book) Disount(c float64) float64 {
	// 注意int和float64类型不一样，
	// 在Go中并不能直接操作，需要进行类型转换
	// 但是float64 可以与字面量值直接相乘... p.price*100不会有问题
	return p.price * c
}
```
对一些方法，传入的参数是值的拷贝而不是地址，经常需要在方法的结尾放回新的结果，可以通过把参数的指针传入，直接修改参数的地址指向的值，常见的io.Write接口就是在指针添加方法。

```
type byteSlice []byte
func (slice *byteSlice) Write(data []byte) (n int, err error) {
	...
}

var b byteSlice
fmt.Fprintf(&b, "this is a string", 7)
```
**以指针或值为接收者的区别在于：值方法可通过指针和值调用， 而指针方法只能通过指针来调用**

## 接口与其他类型/interface ##
Go中的接口为指定对象的行为提供了一种方法：**如果某样东西可以完成这个， 那么它就可以用在这里。**我们已经见过许多简单的示例了；通过实现 String 方法，我们可以自定义打印函数，而通过 Write 方法，Fprintf 则能对任何对象产生输出。

Go中的的接口只是需要实现它的类接受了同名的方法即可，这是很特别的。

### 类型转换 ###
下面的代码：

```
type squence []int
func (s squence) String stirng {
	// do some thing to s
	return ftm.Sprint(s)
}
```
如果忽略类型名，squence和[]int其实是相同的，只不过squence有一个新的类型名而已，所以在他们之间进行类型转换是可以的：

```
is := []int(s)
```
在Go语言中，为了访问不同的方法集会经常进行类型转换。甚至对int和float之间的简单运算都会用到类型转换。

### 接口转换与类型断言 ###
对于混合类型，我们经常需要将一种接口转换为另一种接口，要判断一个值是否实现了某个接口，需要使用到**类型断言**。类型断言接受一个接口值，并从中提取指定的明确的明确类型的值。与其他语言判断类型不同，它的用法如下：

```
var.(typeName)
str.(string)

// 对于使用接口定义的值，可以使用下面的方式
// 其中iv是用 var iv interface{} 定义的，Milk是一中类型
switch str := iv.(type) {
case string:
  fmt.Println(str)
case Milk:
  fmt.Println("milk", str)
default:
  fmt.Println("interface type:", str)
}
```
这是一个看起来很奇怪的用法，不知道Go中有没有提供想instanceof这样的关键字。

### 通用性/Generality ###
> 这部分还不是很了解

若某种现有的类型仅实现了一个接口，且除此之外并无可导出的方法，则该类型本身就无需导出。 仅导出该接口能让我们更专注于其行为而非实现，其它属性不同的实现则能镜像该原始类型的行为。 这也能够避免为每个通用接口的实例重复编写文档。

在这种情况下，构造函数应当返回一个接口值而非实现的类型。

## 接口和方法 ##
Go中几乎任何类型(**除了指针和接口**)都能添加方法（添加的示例已经看过很多了，就不赘述了），因此几乎任何类型都能满足一个接口（即使是基于int的类型也能用来满足一个特别的接口），常见的用法就是**任何实现了ServeHTTP方法的对象都能处理HTTP请求**。

> 甚至可以为func添加方法，当然需要先将函数定义为一种新类型

Handler接口定义如下：

```
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```
下面是一个实现该接口的示例：

```
import "net/http"

type milkServer struct {
	n int
}
// 注意参数中的http不能少，参考Go的包机制
func (m *MilkServer) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	m.n++
	// do some thing
	fmt.Fprintf(w, "request count at %d\n", m.n)
	...
}

// 添加绑定
m := new(Milk)
http.Handle("/count", m)
// 也可以直接绑定一个与ServeHTTP签名相同的函数
func ArgsFunc(w http.ResponseWriter, req *http.Request) {
	...
}
http.HandleFunc("args", ArgsFunc)
```
当然也可以绑定到一个信道上面，然后使用信道新更新一些内部状态：

```
type Chan chan *http.Request
func (c Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	c <- req
	fmt.Fprint(w, "request received")
}
```
