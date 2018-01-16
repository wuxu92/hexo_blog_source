---
date: 2015/07/27 12:34:50
title: Go by Example 中值得注意的点(一)
categories:
- programming
tags:
- golang
- go
---

## 不定参数 ##
不定参数的用法已经知道了，函数可以定义如下：

```golang
func sum (nums ...int){
	total := 0
	for _, num := range nums {
		total += num
	}
	return total
}
```

这样可以计算任意个传入的int参数的和，但是有下面的用法：

```
nums := []int{1,2,3,4,5}
t := sum(nums...)
```
`nums...` 这样的用法在之前没有遇到过，它可以用来表示， 将一个slice的元素作为函数的实参列表。

## 闭包 ##
对于使用过js的人，闭包是一个非常熟悉的概念了。js使用闭包实现内部变量的私有化，go中也有同样的目的？
与js不同的是在定义函数的地方要把返回值写明返回的函数的签名。

```golang
func intSeq() func() int {
	i := 0
	return func() int {
		i += 1;
		return i
	}
}
```
这样定义的intSeq函数调用后的返回值就是一个函数。

```golang
next := intSeq()
// next是一个命名函数了
next()
```
相比js的闭包，功能不是那么丰富，不过本质差不多。

## 指针 ##
Go提供了指针支持，允许传递值的引用。函数的参数可以是某种类型的指针。比如：

```golang
func zeroptr(i *int) {
	*i = 0;
}

i := 1
zeroptr(&i)
```
对于指针要理解星号（*）符号和引用符号（&）的作用。

> Assigning a value to a dereferenced pointer changes the value at the referenced address.

语法 `&i`得到的是`i`的内存地址，也就是i的指针。也可能存在指向指针的指针。在函数体内部，`*i`将指针/内存地址 *解引用(dereferendes)*为地址指向的值。所以赋值语句为`*i=0`这中形式。

## 结构体 ##
go中使用关键字struct定义结构体，和C语言的结构体非常相似，不过相比C语言要简单很多了，没有C语言那些复杂的指针维护。
结构体可以使用字面量初始化：

```golang
// 定义一个结构
type Person struct {
	name string
	age int
}

me := Person{"wuxu", 22}
anotherMe := Person{name:"wuxu", age:22}
mePtr := &me
```
还是要注意指针的理解， me是一个对象， &me返回的是对象的地址，复制给 mePtr， 如果要将mePtr的内存存放新的对象应该如下:

```golang
*mePtr = Person{"new Name", 33}
// 错误: mePtr =  Person{"new name", 33}
```
修改了mePtr的对象，me的对象也被修改了，因为他们是同一片内存区域。

## 方法（method） ##
我们知道在Go中可以给struct“绑定”方法，其实现方式如下:

```golang
func (p Person) say(words string) {
	fmt.Println(p.name, "says: ", words)
}

func (p *Person) run(d int) {
	fmt.Println(p.name, " runs away ", d, " meters")
}
```
上面给Person结构体绑定了两个方法，看以看到，方法绑定中，一个使用 `Person` ,一个是 `*Person`他们有什么区别呢？
在方法调用时，Go会自动处理变量和指针的转换，即下面的调用时正确的：

```
me := Person("wuxu", 22)
me.say("wtf")
me.run(100)
```
虽然run方法的参数类型是 `*Person`，使用me调用也能正确执行，那么在绑定参数时的`*`有什么作用？ 这里其实和值与引用相似。不带`*`的参数相当于值传递，带`*`的相当于引用传递，所以在say方法中对p的属性修改不会改变调用者的值，而在run方法中修改p的属性会表现在调用者上。

> Go automatically handles conversion between values and pointers for method calls. You may want to use a pointer receiver type to avoid copying on method calls or to allow the method to mutate the receiving struct.

但是要注意，如果使用 `*`的话，方法便不再是绑定在Person结构体上了，而是在`*Person`上了。
虽然Person的对象也能调用，这在接口实现中会有表现： 如果一个接口定义了run方法，那么实现这个接口的struct绑定方法时，不能使用`*`
## 接口 ##
Go的接口，只要定义的类型“绑定”了接口中定义的方法，就认为类型实现了接口。没有implements这类的显式声明实现接口。

一个接口实现的示例：

```golang
type animal interface {
  run(int)
}

type Pig struct {
  name string
  age int                                    
}
func (p Pig) run(d int) {
  fmt.Println("Pig runs away ", d)
}

func runAway(an animal) {
  an.run(100);
}                                  
func inters() {
  runAway(Person{"TM", 22, false})
  runAway(Pig{"wangcai", 1})
}
```
关于接口的更多内容可以查看 [这篇博客](http://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go "http://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go")


## 错误 ##
我们知道Go没有异常系统，而是通过一个显式声明的独立返回值（利用多返回值特性）来声明错误。一般在最后一个返回值声明错误。
error是系统提供的返回错误接口，可以自己定制类型实现该接口作为定制的错误类型返回。如果错误位返回nil表示没有错误。

```golang
func (e myErr) Error() string {
  return fmt.Sprintf("%d - %s", e.arg, e.prob)
}
func (p Person)buy(item string) (bool, error) {
  if p.age < 18 {
    return false, &myErr{0, "too yong"}
  }
  return true, nil
}
```
常见的自定义错误如上，注意几点，在返回处， 需要使用 `&myErr`，否则会报类型错误。如果没有自定义错误，可以直接使用内置的错误类型

```
return false, errors.New("too yong")
```
注意，需要导入errors包。
使用自定义错误的好处是可以在调用出对错误做类型判断，已进行相应的处理。


```
_, e := me.buy("shirt")
if me, ok := e.(*myErr); ok {
	// with myErr return, me ref to return error
	...
}
```

## 待续 ##

见第二部分


