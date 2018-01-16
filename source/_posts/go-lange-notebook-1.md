---
date: 2015/10/22 12:34:50
title: Golang 学习笔记： 指针,函数,内嵌及接口
categories:
- programming
tags:
- go
- golang
- lang
---
## 指针和值 ##
自定义类型的实例化对象有值类型和指针类型的区别（个人理解），比如下面：

```golang
type Unicorn struct {
  Name string
  Age int
  weight int
}
larry := new(Unicorn)  // *Unicorn, pointer type
larry := &Unicorn{}	   // 这和上面一样 

la := Unicorn{}		// value type
```

## 定义函数 ##
定义函数很简单：

```golang
func message(name string) string {
	message := fmt.Sprintf("hello, %v", name)
	return message
]
```
<!-- more -->
这是最常见的声明方式，我们还可以这样：

```golang
func message(name string) (message string) {
	message = fmt.Sprintf("hello, %v", name)
	return
}
```
使用命名的返回值，可以在return部分生命变量名。

## 接口类型 ##
接口类型是`interface{}`，在声明函数时指定相应参数类型如下：

```golang
func print(v ...interface{}) {
	...
}
```
注意其中的大括号。这里还是用了可变参数，在参数类型前面的三个点号表示这是一个数量可变的参数，可变参数只能在参数列表的最后。这样接收到的v是一个切片，可以对其进行range之类的操作。

关于接口类型的类型判断，之前提到过golang的类型断言使用`x.(T)`的方式。

在golang中的强制类型转换，尤其是将interface{}转换为其他类型的时候，需要做类型断言，否则会报类似下面的错误：

```
vp := (Person)p  // p 是interface{}类型

报错:
.\variable_param_test.go:15: cannot convert p (type interface {}) to type Person: need type assertion
```
要把p转换为Person对象，应该如下：

```golang
func convert(v ...interface{}) {
	for _, p := range v {
		if t, ok := p.(Person); ok {
			// 现在t就是转换后的Person对象了
			fmt.Println("person", t.name)
		} else {
			fmt.Println("not person", p)
		}
	}
}
```

## 为类型绑定方法 ##
这部分也将过了，在对自定义类型绑定方法时的语法如下：

```golang
type Message struct {
	x string
	y *string
}

func (m *Message) Update(x, y string) {
	m.x = x
	m.y = &y
}

func (m Message) String() string {
	var y = "nil"
	if m.y != nil {
		y = *m.y
	}
	return fmt.Sprintf("%s, %s", m.x, y)
}
```

需要注意的是， Message和*Message是两种不同的类型。上面的代码为*Message类型绑定了方法`Update(x, y string)`，而为类型Message绑定了方法`String()`
当然也可以这样理解：对于需要修改类型内容，并且方法调用结束要保留这些修改的方法使用带指针的绑定。因为在调用时Go会自动对指针进行“非常乐观”的转换，实际上调用这些方法的类型并没有严格要求，也就是，对上面的绑定的方法可以如下调用：

```
m := Message{}  // 或者 m := &Message{} 也是一样的
m.Store("hello", "world")
s := m.String()
```

## Go的类型嵌入及其方法调用 ##
Go没有提供面向对象的继承机制，意味着熟悉的子类调用父类方法的思维不再适用了。
但是Go提供了一种类似于Javascript的原型继承很相似的方式，叫做"类型内嵌"，即A类型内嵌按一个B类型的变量，可以直接在A类型的变量调用B类型绑定的方法，只要方法没有冲突。

请看下面的例子：

```golang
type ColorMessage struct {
	Message
}

func TestTypeEmbed(t *testing.T) {
	cm := ColorMessage{Message{"colorful", nil}}
	cm.StorePtr("new", "Message")  // 直接调用Message绑定的方法
	fmt.Println(cm)     // 会调用Message的String()方法
}
```

更复杂一点，多个内嵌类型都有Stirng方法，会依次调用它们的String方法：

```golang
type Ring struct {
	ring string
}
func (r Ring) String() string {
	return r.ring
}

type Bell struct {
	bell string
}
func (b Bell) String() string {
	return b.bell
}

type ColorMessage struct {
	Message
	Ring
	Bell
}
func TestTypeEmbed(t *testing.T) {
	cm := ColorMessage{Message{"colorful", nil}, Ring{"morning"}, Bell{"dida, dida"}}
	cm.StorePtr("new", "Message")
	fmt.Println(cm)  // 输出: {new, Message morning dida, dida}
}
```

> Go allows **user-defined** types to declare methods on either a value type or a pointer to a value type. When methods operate on a value type the value manipulated remains **immutable** to the rest of the program (essentially the method operates on a copy of the value) whilst with a pointer to a value type any changes to the value are apparent throughout the program. 

注意其中的user-defined types,只有对用户定义类型是这样的，对于切片和数组并不适用。

## 接口 ##
在Java，C#这些传统的面向对象语言中，接口代表着一种类型约束。子类需要显示声明实现了某些接口，以确保他们拥有某种能力，或者具有某种类型。在Golang中的接口定义了一些方法，只要实现了这些方法就认为类型实现了该接口。

这与以往的类型系统很不一样。并且对于接口系统来说，方法绑定的对象带不带指针很重要了；这里Go并不会自动对指针做一些转换。如下,定义了一个Singer的接口，后面定义的两种类型Sparrow：和Dove，分别将`Sing()`方法绑定在其类型和指针类型上，那么就是`Sparrow`和`*Dove`二者实现了接口`Singer`,而Dove并没有。

```golang
type Singer interface {
	Sing() string
}

type Sparrow struct {}
func (s Sparraw) Sing() string {
	return "i am a sparrow"
}

type Dove struct {}
func (d *Dove) Sing() string {
	return "ge..ge..ge.."
}

type Zoo struct {
	s, d Singer
}

func (z Zoo) CanSing() (ok bool) {
	if _,ok = z.s.(*Sparrow); !ok {
		_, ok = z.d.(*Dove)
	}
	return
}

func TestZoo(t * testing.T) {
	z := Zoo{}
	fmt.Println(z.CanSing())
	z.s = new(Sparrow)
	fmt.Println(z.s.Sing())
	fmt.Println(z.CanSing())
	
	z.d = Dove{}  // 这样会报错，因为Dove没有实现接口
}
```
理解最后一行 `z.d=Dove{}`会在编译期报错，就是因为Dove并没有实现Singer接口，不能保存在使用Singer定义的变量中，如下两种方式都可以:

```golang
z.d = new(Dove)
z.d = &Dove{}
```
和类型内嵌一样，接口也可以通过内嵌接口来组合接口的方法。最常见的标准io库中的ReadWriter接口：

```golang
type Reader interface {
	Read(p []byte) (n int, err error)
}

type Writer interface {
	Write(p []byte) (n int, err error)
}

type ReadWriter interface {
	Reader
	Writer
}
```

## 初始化 ##
之前也提到过，Go的包可以包含一个或者多个init方法。这些方法会在程序初始化的时候执行。这是Go很有特色的语言特性。init方法可以包含任意的go代码，甚至勀把所有的代码都扔到init里面，而只留下一个空的main方法～～

因为一个包内可以定义多个init函数，并且它们的执行顺序并不是固定的，所以不要使代码逻辑依赖于init方法的执行顺序。

```golang
func init(){
	fmt.Println("inited")
}
// 在一个.go文件中定义多个init方法是允许的
func init() {
	fmt.Println("init again")
}
```

下一部分网络的新开一篇
