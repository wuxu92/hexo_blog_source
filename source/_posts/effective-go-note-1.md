---
date: 2015/07/18 12:34:50
title: Effective Go笔记：函数，数据，初始化
categories:
- programming
tags:
- golang
- effective
---

effective go 中的部分笔记，这是第一部分 


部分笔记摘要,参考： [https://go-zh.org/doc/effective_go.html](https://go-zh.org/doc/effective_go.html "https://go-zh.org/doc/effective_go.html")


## 函数 ##
Go的函数有一个很特别的性质那就是多值返回，这样做有一个很方便的地方是可以把错误值返回，这样做和后面会讲到的取消异常也是有关的。


一个Go函数的签名可能是：

```
func (file *File) Write(b []byte) (n int, err error)
```
这是一个典型的Go函数签名，初看起来比较复杂，一点点分析：

1. func关键字，类似php中的function
2. (file *File) 表示这是一个添加到*File接口上的方法。这也是Go比较有意思的地方，通过*附着*接口方法的方式实现接口
3. Write 很简单就是方法名了，大写开头的方法表示共有的导出方法，如果是小写的方法名是不能导出的
4. (b []byte)方法的参数，形参名为b，类型为[]byte
5. (n int, err error) 返回类型，如果是多指返回则使用小括号括起来，如果是一个返回值就不需要了。比较有意思的是返回值也可以作为有命名返回值，比如这里的n和err，这样表示把函数作用域内的n和err变量返回，这样返回语句只需要写return，而不需要显式说明具体变量

### Defer ###
Defer 是Go的关键字，用于预设一个函数调用，一个使用Defer调用的函数会**在执行defer的函数返回之前**立即执行，常用来释放资源，比如解锁互斥和关闭文件
使用defer推迟的函数会按照**后进先出**的顺序执行。
<!-- more -->
## 数据/data ##
Go提供new和make两个内建的函数来分配内存
### new ###
new是一个用来分配内存的内建函数，和其他函数中的new实例化对象不一样，new在Go中只会将内存置零，而不会初始化内存。new(T)只会为T的新项分配内存并返回地址

### 构造函数与复合字面量 ###
Go中的字面量非常强大，可以使用字面量来初始化各种类型～

```
type person struct {
	name string
	age int
}

me := person{"wuxu", 22}
```
可以在函数中返回一个局部变量的引用/地址。局部变量的数据在返回后仍然有效。如果类型不包含字段，则new(Person)和&person{}是等价的

```
a := [...]string   {"no error", "Eio", "invalid argument"}  // array
s := []string      { "no error", "Eio", "invalid argument"}  // slice
m := map[int]string{Enone: "no error", Eio: "Eio"}  // map
```

### make ###
内建函数make可以用来创建切片，映射和信道（channel），并返回类型为T（不是*T）的一个已经初始化（而非置零）的值。make 用于初始化其内部的数据结构并准备好将要使用的值

```
make([]int, 10, 100) //不要这样用，因为会看不懂。。。
make([]int, 0, 10) 
// allocates a slice of length 0 and capacity 10.
```
这样会分配一个具有100个int的数组空间，接着创建一个长度为10，容量为100并指向该数组前10个元素的切片.。有点拗口
注意：make只适用于映射，切片和信道，并且不返回指针，，若要获取明确的指针需要使用new分配内存。
### 数组 ###
数组作为切片的基础构件，要理解切片的工作机制必须对数组非常熟悉。Go的数组和C的数组有很大的区别：

1. 数组是一个值，而不是引用。将一个数组赋予另一个数组会复制其所有元素
2. 若将某个数组传入某个函数，将会收到数组的一份副本而不是指针
3. 数组的大小是其类型的一部分，即[10]int 和 [11]int 是两种类型

**注意** ```var a []int```这样的声明并不是声明一个数组，而是一个切片，```var arr [4]int```这样才是定义了一个数组~

### 切片 ###
切片是对数组的封装，为数据序列提供更加通用的而方便的接口。切片在动态编程语言中也非常常用。
切片保存了对底层数组的引用。切片使用引用传递。
切片的容量在底层数组的范围内是可变的，切片的容量使用内建函数cap获得，切片长度使用内建函数len获得。

#### 二维切片 ####
二维切片有两种方法进行初始化，一种是先初始化顶层切片，再分别对每一行分配切片，一种是一次性分配(make)所有空间，再使用分片操作切分到各个行

```
picture := make([][]int8, 10)
for i := range picture {
  picture[i] = make([]int8, 10)
}
picture2 := make([][]int8, 10)
pixels := make([]int8, 10*10)
for i:= range picture2 {
  picture2[i], pixels = pixels[:10], pixels[10:]
}
```

### 映射/map ###
映射是非常强大的内建数据结构，可以用来关联不同的数据类型，键可以是任何支持相等性操作的类型，比如int, float, string, pointer,interface, struct, araay。不过**切片和映射不能用作键**，因为他们不支持相等性操作。
映射的定义语法：

```
var weekDay = map[int]string {
	1: "monday",
	2: "tuesday",
	3: "wednesday",
	4: "thursday",
	5: "friday",	// 注意最后的,不能省略
}
```
对映射的访问类似于动态语言，使用键作为下标访问

```
weekDay[1]
// 判断某一项是否存在,使用‘逗号ok’惯用法
// 如果ok为false，表示不存在，val置零，如果为true，表示存在
val, ok = weekDay[4]
if ok {
}
```
可以使用内建函数delete删除映射的键值对

```
delete(weekDay, 1)
```

### 打印/print ###
Go提供和C语言类似的格式化打印函数printf族，不过功能更加丰富。这些函数在fmt包中，fmt也是最常用的包之一。

```
// 一下输出等价
fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))
```
常用的占位符： %d, %x, %v, $f, %q ; %v是通用格式，v代表value。*改进的格式 %+v 会为结构体的每个字段添上字段名，而另一种格式 %#v 将完全按照Go的语法打印值*

**若你想控制自定义类型的默认格式，只需为该类型定义一个具有 String() string 签名的方法。**这和java中的toString()功能相似。

```
// 下面的String方法会导致递归， 需要对m使用类型转换
type MyString string
func (m MyString) String() string {
	return fmt.Sprintf("MyString=%s", m) // 错误：会无限递归。转换为string(m)即可
}
```
### 关于append的补充 ###
append函数的签名是：

```
func append(slice []T, item ...T) []T
```
append 会在切片末尾追加元素并返回结果。我们必须返回结果， 原因是层数组可能会被改变。可以发现append方法的签名中使用类型占位符T，但是我们Go程序中并不能使用泛型。。。。

## 初始化/Initialization ##
### 常量/constants ###
Go中常量也是不变量，在编译时创建，即便他们是函数中定义的局部变量也是这样。常量只能是数字，字符，字符串，布尔值。常量定义中不能有函数调用。
枚举常量使用枚举器**iota**创建。使用它很容易创建复杂的值的集合。一个官方示例：

```
type ByteSize float64

const (
    // 通过赋予空白标识符来忽略第一个值
	// 这里这样做好像有问题。。关于iota再写吧，这里不深入了
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```
### 变量/vaiables ###
可以使用var()声明多个变量：

```
var (
 home = os.Getenv("HOME")
 user = os.Getenv("USER")
)
```
### init函数 ###
每一个源文件可以定义一个无参数的init函数来设置一些状态。甚至每个文件**可以有多个init函数**。init执行完才是初始化(包/文件)的结束。常用init函数来检查或者校正程序的状态。

```
func init() {
	// do some thing here
}
```
