---
date: 2015/10/26 12:34:50
title: Go 反射
categories:
- programming
tags:
- go
- golang
- lang
- reflect
---

反射是提供一种程序在运行时检查变量结构，状态的能力，它是元编程的一种形式(metaprogramming)。反射功能强大，但是很多时候反射使得代码难以理解，并且性能并不很好。

## 类型和接口 ##
反射功能是基于类型系统(type system)，所以要先对Go 的类型有足够的认识。我们都知道Go是静态类型的，即使是像`type MyInt Int` 这样定义的类型，虽然MyInt和Int有相同的基础类型(underlying type),但是MyInt和Int还是不一样的,不能互相赋值。

接口是一类重要的特殊类型，它代表的是一系列方法，一个接口声明的变量可以用于存储任何实现了该接口方法的变量。常见的接口如io.Reader和 io.Writer 。

```golang
var r io.Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
```
在使用中，经常需要明确知道r中到底存储的是什么类型的变量(concrete value)。由于Go是静态类型的，r的类型永远是 `io.Reader`。
<!-- more -->
更多时候，直接使用接口 `interface{}` 定义变量来存储任意类型的值。 interface也是静态类型的，interface的变量永远是同样的静态类型，即使在运行时它存储的内容可能改变，但是它的类型还是interface的。

## 接口的表示 ##
interface类型的变量存储了一个元组(a pair): 一是赋值给这个变量的具体值(concrete value),一是那个值的类型描述符(type descriptor)。更准确的说，值是实现了接口的具体的数据元素，类型描述了元素的完整类型。例如：

```golang
var r io.Reader

tty, e := os.OpenFile("/dev/tty", os.O_RDWR, 0)

if e != nil {
	return nil, e
}
r = tty
```
这段代码里，可以这么理解，r包含一个(value, type)对，那就是(tty, *os.File)。注意 *os.File 不仅是实现了Read接口。虽然r只能访问 Read 方法，但是value部分仍然是保留了 *os.File 类型的所有信息的，所以我们可以做如下的操作：

```golang
var w io.Writer
w = r.(io.Writer)
```
这里r可以类型转换为一个 io.Writer ,这是因为r中保存的具体值类型 *os.File 也实现了 io.Writer 的方法。现在w也保存了(tty, *os.File)这个数据对。静态类型的接口来决定在运行时接口变量调用什么方法，即使其具体值包含很多其他的方法。

我们当然也可以把它们再赋值给一个空接口(interface{})

```
var empty interface{}
empty = w
```
empty也会存储(tty, *os.File)。很容易理解，空接口可以用于存储任何类型的值，并且保留该值的所有类型信息。

> One important detail is that the pair inside an interface always has the form (value, concrete type) and cannot have the form (value, interface type). Interfaces do not hold interface values

## 反射第一定律 ##
**反射是从接口值到反射对象**

简单地说，反射只是一种检查存储在接口变量中的值和类型信息对的机制。Go的反射主要在 `reflect` 包中，reflect包中两种类型 `Type` 和 `Value`。 这两个类型提供对对接口变量内容的访问。具体是使用 `reflect.TypeOf` 和 `reflect.ValueOf`两个方法。

```golang
import (
	"fmt"
	"reflect"
)

func main() {
	var x float64 = 3.14
	fmt.Println("type", reflect.TypeOf(x))  // output type float64
	fmt.Println("value", reflect.ValueOf(x))
}
```
实际上 Value 类型有一个 Type 方法也可以返回 reflect.Value 的 Type。同时Value和Type类型都提供了一些方法进行一些操作。比如 `Kind()` 方法一个反类的类型

```golang
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())
fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
fmt.Println("value:", v.Float())
```
输出

```
type: float64
kind is float64: true
value: 3.4
```
为了使API简便，反射库会进行一些类型转换，比如int类型等会用最大的int64保存，如下

```
var x uint8 = 'x'
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())                            // uint8.
fmt.Println("kind is uint8: ", v.Kind() == reflect.Uint8) // true.
x = uint8(v.Uint())                                       // v.Uint returns a uint64.
```
然后要去人静态类型(static type)和基础类型(underlying type)。reflect.ValueOf的 Kind()方法返回是静态类型，比如 `type MyInt int` 的反射使用Kind返回的是`reflect.Int`,MyInt 是其静态类型

## 反射第二法则 ##
**从反射对象到接口值**从一个reflect.Value变量，使用Value的Interface方法返回到 接口值。方法签名如下

```
func (v Value) Interface() interface{}
```
`Interface()`可以看作ValueOf的逆方法。除了返回接口的静态类型变为了interface{}。

## 发射第三法则 ##
**要修改一个反射对象(reflection object), 其值必须是可设置的(settable)**

先看一段错误但是值得一看的代码：

```
var x float64 = 3.4
v := reflect.ValeOf(x)
v.SetFloat(7.1)
```
如果试图运行这段代码，会得到如下报错： `panic: reflect.Value.SetFloat using unaddressable value` 这并**不是**因为 7.1 这个值是不可寻址的(unaddressable)，而是 v 是不可设置的(not settable)。 Settability是反射 Value 的一个属性，并不是所有的反射 Value 都拥有这个属性。可以用`CanSet()`方法查询。对上面的v

```
fmt.Println("settability of v ", v.CanSet())  // prints: settability of v false
```
那么该如何理解可设置性呢?可设置行可以理解为一种更严格的可循执性。表示反射对象能修改实际存储。可设置性由反射对象是否拥有原始元素(hold the original item)来决定。

对上面的x，反射是直接传递以各x的拷贝过去，并不是x本身，所以 `v.SetFloat(7.1)` 要更新 x 的值，但是v只是从x产生的，并不是x本身。所以并不运行进行这样的操作。

这和方法参数的值参数和引用参数差不多。所以可以使用引用的ValueOf 来获得可设置性。

```golang
var x float64 = 3.4
p := reflect.ValueOf(&x) // Note: take the address of x.
fmt.Println("type of p:", p.Type())  // *float64
fmt.Println("settability of p:", p.CanSet())   //这里仍然是 false
```
事实上我们也不是要设置p，而是 *p, 我们需要调用Elem()方法

```golang
v := p.Elem()
fmt.Println(v.CanSet())  // 这里就是true了
```
这里x就是x了，于是可以 `v.SetFloat()` 设置x的值了。

## Structs ##
更多的时候是对struct使用反射。下面的代码遍历所有的域(field)

```golang
type T struct {
    A int
    B string
}
t := T{23, "skidoo"}
s := reflect.ValueOf(&t).Elem()  // 注意 & 取地址符
typeOfT := s.Type()
for i := 0; i < s.NumField(); i++ {
    f := s.Field(i)
    fmt.Printf("%d: %s %s = %v\n", i,
        typeOfT.Field(i).Name, f.Type(), f.Interface())
}
```
输出

```
0: A int = 23
1: B string = skidoo
```
struct的属性(field)都是可设置的，因为反射对象是可设置的(是一个指针)。

```
s.Field(0).SetInt(77) // t.A = 77
s.Field(1).SetString("Sunset Strip")  // t.B = "Sunset Strip"
```

> If we modified the program so that s was created from t, not &t, the calls to SetInt and SetString would fail as the fields of t would not be settable.

## 总结/结论 ##
Here again are the laws of reflection:

Reflection goes from interface value to reflection object.
Reflection goes from reflection object to interface value.
To modify a reflection object, the value must be settable.
Once you understand these laws reflection in Go becomes much easier to use, although it remains subtle. It's a powerful tool that should be used with care and avoided unless strictly necessary.

There's plenty more to reflection that we haven't covered — sending and receiving on channels, allocating memory, using slices and maps, calling methods and functions — but this post is long enough. We'll cover some of those topics in a later article.

**By Rob Pike**


参考

- [http://blog.golang.org/laws-of-reflection](http://blog.golang.org/laws-of-reflection "http://blog.golang.org/laws-of-reflection")
