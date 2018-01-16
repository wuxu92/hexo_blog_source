---
date: 2015/08/26 12:34:50
title: Go by Example(五)：杂烩1
categories:
- lang
tags:
- golang
- tricks
---

## 排序包/sort ##
go内置的sort包提供了内置类型和用户自定义类型的排序功能。需要注意的sort包的方法对传入的参数排序后直接修改参数，而不是返回一个新的数组。可以这么理解：一般传入排序的是一个数组，数组一般作为引用传递，而对于引用传递的修改直接反映在原数组上。
sort包提供对数组排序的函数，函数名为待排序数组元素类型加s，参数为待排序数组，比如

```
strs := []string{"bcd", "abk", "opq", "hij"}
sort.Strings(strs)
// strs has been sorted
sort.StringsAreSorted(strs) // true
```
同样对于int数组的排序方法为`sort.Ints(ints)` sort包还提供了一个判断数组是否排序的系列方法，其方法名为参数类型加s加AreSorted(); 比如 `IntsAreSorted(ints), StringsAreSorted(strs)`

## 给自定义类型排序 ##
类似于java与其他面向对象语言可以为实现了sortable接口的对象数组进行排序，go也提供了为自定义类型数组进行排序的方法。`sort.Sort(myArray)`中myArray是一个自定义的类型，它是某种类型的数组类型。比如:

```golang
type myType []string
```
要实现排序，也需要为这种类型绑定一些方法，就像实现sortable接口的方法一样。实际上绑定这些方法也是实现go内置定义的 [sort.Interface](https://golang.org/pkg/sort/#Interface "https://golang.org/pkg/sort/#Interface"), 要实现Len（） int, Swap和Less() bool三个方法。
<!-- more -->
实际上Go内置的sort.Sort()方法使用了泛型实现，虽然go语言本身不支持泛型。


```
func (m myType) Len() int {
  return len(m)
}
func (m myType) Swap(i, j int) {
  m[i], m[j] = m[j], m[i]
}
func (m myType) Less(i, j int) bool {
  return len(m[i]) < len(m[j])
}
```
这样让myType类型实现了sort.Interface接口。可以用sort.Sort()来排序了。

```golang
fruits := []string{"kiwi", "apple", "plum"}
sort.Sort(myType(fruits))
```
这里也算是一种编程pattern，用来实现排序。

## panic/快速失败 ##
panic是go内置的fail fast功能函数，当程序运行到某种异常状态不能处理的时候使用panic函数结束程序运行。

> A common use of panic is to abort if a function returns an error value that we don’t know how to (or want to) handle. Here’s an example of panicking if we get an unexpected error when creating a new file.

```
_,err := os.Create("/tmp/file.cc")
if err != nil {
	panic(err)
}
```

> Note that unlike some languages which use exceptions for handling of many errors, in Go it is idiomatic to use error-indicating return values wherever possible.

## defer ##
defer是延迟执行，这在js异步编程中也是一个很重要的概念。更准确的说，defer可以保证一个函数调用在程序执行结束前执行。应为这个函数会在函数结束前执行，所以可以用来执行一些清理操作。
所以在打开文件，数据库连接等操作后可以用defer调用一个文件关闭，资源释放的函数。一个简答的示例：

```
func defers() {
  f := os.Create("/tmp/defer.txt")
  defer func(f *os.File) {
	f.Close()
  }(f)  // 此函数在整个调用结束时调用
  fmt.Fprintln(f, "defer close file")
  ...  // do others here
}
```

## collection ##
由于Go不提供泛型/generic支持，所以go库并没有像java一样原生提供所有类型的集合/数组都可以使用的一些常用函数，比如Index(), Include(), 如果我们自己定义的类型结合需要这些函数，就需要对自己的类型进行编程。

```
func Index(vs []string, needle string) int {
  for i:=0; i<len(vs); i++ {
    if vs[i] == needle {
       return i
    }
  }
  return -1
}
```
这是一个为string数组查找是否存在某个元素的方法，其他还包括,Include, All, Any, Map， Filters等等。
由于没有泛型的支持，现在集合操作变得非常的麻烦，必须为所有的类型编写方法。不过值得期待的是Go开发组表示会在之后的合适版本加入对泛型的支持。希望这一个版本早一点到来吧。

## 字符串处理 ##
go中的字符串主要使用strings包提供的字符串相关函数。 使用前需要先倒入包 `import "strings"`。
strings包提供了很多方法，不过要注意的是这些方法都不是字符串对象本身的方法，而是需要在调用时把字符串作为第一个参数传入。这有一些像面向对象中的静态方法。

除了strings包提供的方法外，常用的是len()获取字符串的长度，还有使用索引获取某个索引位的字符。

```
var p = fmt.Println
p("Contains:  ", s.Contains("test", "es"))
p("Count:     ", s.Count("test", "t"))
p("HasPrefix: ", s.HasPrefix("test", "te"))
p("HasSuffix: ", s.HasSuffix("test", "st"))
p("Index:     ", s.Index("test", "e"))
p("Join:      ", s.Join([]string{"a", "b"}, "-"))
p("Repeat:    ", s.Repeat("a", 5))
p("Replace:   ", s.Replace("foo", "o", "0", -1))
p("Replace:   ", s.Replace("foo", "o", "0", 1))
p("Split:     ", s.Split("a-b-c-d-e", "-"))
p("ToLower:   ", s.ToLower("TEST"))
p("ToUpper:   ", s.ToUpper("test"))

p.("Len", len("length"))
p.("Char of index 2", "index"[2])
```

## 格式化字符串 ##
go提供的fmt.Printf()方法可以用来格式化输出字符串，其格式化的方案与C语言基本相同。使用Sprintf()会将格式化结果以字符串返回而不是打印在屏幕上。

- 如果值是一个结构体，%+v 的格式化输出内容将包括结构体的字段名。
- `%#v` 形式则输出这个值的 Go 语法表示。例如，值的运行源代码片段。
- 需要打印值的类型，使用 `%T`。
- 格式化布尔值是简单的。 `fmt.Printf("%t\n", true)`
- 格式化整形数有多种方式，使用 `%d`进行标准的十进制格式化。
- 这个输出二进制表示形式。 `%b`
- 这个输出给定整数的对应字符。  `%c`
- `%x` 提供十六进制编码。
- 对于浮点型同样有很多的格式化选项。使用` %f `进行最基本的十进制格式化。
- `%e` 和 `%E` 将浮点型格式化为（稍微有一点不同的）科学技科学记数法表示形式。
- 使用 `%s` 进行基本的字符串输出。
- 像 Go 源代码中那样带有双引号的输出，使用 `%q`。
- 和上面的整形数一样，`%x` 输出使用 base-16 编码的字符串，每个字节使用 2 个字符表示。
- 要输出一个指针的值，使用 %p。
- 当输出数字的时候，你将经常想要控制输出结果的宽度和精度，可以使用在 `%` 后面使用数字来控制输出宽度。默认结果使用右对齐并且通过空格来填充空白部分。 `fmt.Printf("|%6d|%6d|\n", 12, 345)`
- 你也可以指定浮点型的输出宽度，同时也可以通过 宽度.精度 的语法来指定输出的精度。 `fmt.Printf("|%6.2f|%6.2f|\n", 1.2, 3.45)`
- 要左对齐，使用 `-` 标志。 ` fmt.Printf("|%-6.2f|%-6.2f|\n", 1.2, 3.45)`
- 你也许也想控制字符串输出时的宽度，特别是要确保他们在类表格输出时的对齐。这是基本的右对齐宽度表示。 `fmt.Printf("|%6s|%6s|\n", "foo", "b")`
- 要左对齐，和数字一样，使用 `-` 标志。 `fmt.Printf("|%-6s|%-6s|\n", "foo", "b")`

## 正则表达式 ##
*待补充*

## JSON ##
*待补充*

## Time ##
前面已经使用过time包的内容了，go的time包提供了很多常用的时间相关方法，具体内容可以查看time的godoc： [http://golang.org/pkg/time/](http://golang.org/pkg/time/ "http://golang.org/pkg/time/") 
time提供Duration类型，其基本类型是int64，用于表示一段时间的长短；time包还提供Location类型用于时区定位，还提供了时间变量格式化输出，并且内定义了一些常用时间格式化标准，用方法:

```
// 创建一个时间
then := time.Date(2015, 8, 27, 0, 0, 0, 0, time.UTC)  // Universal Coordinated Time
// 获取时间的值
then.Year()
then.Month()
then.Day()
then.Hour()
then.Minute()
then.Second()
then.Nanosecond()
then.Location()
then.Weekday()
// 时间运算
now := time.Now()
then.Before(now)
then.After(now)
then.Equal(now)
diff := time.Duration(time.Second * 100)
then.Add(diff)
then.Add(-diff)
```
更多时间相关的方法，查阅godoc即可。

## epoch/时间戳 ##
time包提供了对unix时间戳的支持，在Time变量调用Unix方法即可放回该时间对应于unix epoch的秒数（unix_timestamp）。甚至还能获取毫秒，微秒数据。

```
nowSecond := time.Now().Unix()
// 还可以获取毫秒，微妙数据。
  fmt.Printf("%d-%02d-%02dT %02d:%02d:%02d %04d", now.Year(), now.Month(), now.Day(), now.Hour(), now.Minute(), now.Second(), now.Nanosecond()/1000000)
```



