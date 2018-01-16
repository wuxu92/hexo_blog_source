---
date: 2015/07/20 12:34:50
title: Effective Go笔记：空白标识符，接口检查与内嵌
categories:
- programming
tags:
- golang
- lang
---

effective go 中的部分笔记，这是第三部分 

部分笔记摘要,参考： [https://go-zh.org/doc/effective_go.html](https://go-zh.org/doc/effective_go.html "https://go-zh.org/doc/effective_go.html")

## 空白标识符 ##
空白标识符是指"_"，是一个特殊的变量标识，在之前的for...range 循环中已经使用过。

### 为未使用的变量或者包使用 ###
在Go中如果定义了一个变量没有使用，或者导入一个没有使用的包会在编译时报错，未使用的包会让程序体积变大并拖慢编译速度，初始化了而不是用的变量会留下某种隐患。
但是某些时候我们确实需要丢弃一些没有的变量，比如在for range循环中，可能需要丢弃键或者值，或者程序写到中途某些变量已经定义只是还没有使用等等，这时就需要空白标识符了。

```golang
// 检查某个路径是否存在
if _, err := os.Stat(paht); os.IsNotExist(err) {
	fmt.Println("path not exist")
}
```
对于未使用的变量和未使用的导入包，可以把它们先赋值给空白标识符"_"关闭编译器的错误，在使用了这些变量后再删掉这些代码：

```golang
fd = 32
....
_ = fd
var _ = fmt.Printf  // 用于调试使用，结束时删除
```

## 接口检查 ##
如果只需要判断某个类型是否实现了某个接口而不需要实际使用接口本身，可以使用空白标识符：

```
if _, ok := val.(Milk); ok {
	fmt.Printf("value %v has implement interface milk", val)
}

// 接口检查示例
type iMilk interface { 
  getName() string
}
type Milk struct { 
  name string
}
func (m Milk) getName() string {
  return m.name
}
func main() {
  var m interface {}
  m = Milk{"yili"}
  if _,ok := m.(iMilk); ok {
    fmt.Printf("value %v has implement iMilk\n ", m)
  } else {
    fmt.Println("has not implememt")
  }
}

```

## 内嵌/Embedding ##
Go语言并不提供典型的类与继承系统，所以也没有子类化的概念。继承是一个重要的概念，但是很多时候更推荐的做法是使用组合，Go提供的内嵌和组合概念差不多，可以将类型**内嵌**到结构体或者接口中，实现某种组合功能。

一种典型的内嵌的ReadWriter接口的定义：

```golang
type Reader interface {
	Read(p []byte) (n int, err error)
}
type Writer interface {
	Write(p []byte) (n int, err error)
}
// 一个结合了Reader和Writer的接口
type ReadWriter interface {
	Reader
	Writer
}

// 也可以内嵌到一个结构体中
type ReadWriter struct {
	r *Reader
	w *Writer
}
```

内嵌的接口一般需要实现被内嵌接口的方法，满足其需求，比如要提供Read方法：

```golang
func (re *ReadWriter) Read(p []byte) (n int, err error) {
	return rw.r.Read(p)
}
```
而通过直接内嵌结构体，我们就能避免如此繁琐。 **内嵌类型的方法可以直接引用**。
当内嵌一个类型时，该类型的方法会成为外部类型的方法， 但当它们被调用时，该方法的接收者是内部类型，而非外部的。在我们的例子中，当 bufio.ReadWriter 的 Read 方法被调用时， 它与之前写的转发方法具有同样的效果；接收者是 ReadWriter 的 reader 字段，而非 ReadWriter 本身：

```
type Logger struct {
  content string
}
func (l Logger) Log(s string) {
  l.content += s
  fmt.Println(s)
}
type infoLog struct {
  count int
  *Logger
}
func main() {
	var logger = infoLog{0, &Logger{"sss"} }
	// 可以直接调用内嵌类型的方法
	logger.Log("embedding function invoke")
}
```
内嵌类型可能会引起命名冲突的问题。更深层次的命名字段或方法会被覆盖。若相同的嵌套层级上出现同名冲突，通常会产生一个错误。但是如果冲突的名字不会被使用就没有问题......

*并发部分有点多，单独做一篇吧*
