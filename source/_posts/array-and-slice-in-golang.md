---
date: 2015/10/13 12:34:50
title: Golang中array和slice的总结
categories:
- programming
tags:
- golang
- go
---

golang内置对切片(slice)的支持，但是golang的类型系统对数组(array)的使用并没有动态/脚本语言那么方便，在使用中经常需要使用切片和数组的一些操作。

首先对于数组要注意数组的数据类型是存储的**元素的类型**加**数组的长度**；也就是说`[2]int`和`[3]int`是两种不同的类型。而切片的类型只和**数据的类型**有关。

声明一个数组和一个切片是不同的。

```golang
var arr [5]int
arr := [5]int{1,2,3,4,5 }

var sli []int
sli := []int{1,2,3,4,5}
```
<!-- more -->
数组的类型是一维的，可以构造数组的数组来实现多维数组。
一般切片使用make方法创建，make方法的签名如下

```
func make([]T, len, cap) []T
```
T是声明的切片保存的数据的类型。其中len表示切片的长度，cap表示capacity。一般使用时省略cap参数，默认和len参数相同。capacity和length在切片中是可能不一样的。

```
var s []int
s = make([]int, 5)
// equals to 
s := []int{0,0,0,0,0}
```
切片类型重要的操作就是“切片”，使用一个**半开的区间**，使用两个索引指定要切片的范围([lo, hi]两个索引是前闭后开的)。

```
s := []byte{'a', 'd',  's', 'f', 'g', 'h'}
// s[1:4] == []byte{'d', 's', 'f' }
```
使用时要注意前闭后开的性质，切片后的长度为`hi-lo`。

特别需要注意的是： 切片操作返回的切片与原切片**共享存储**。切片操作不不会复制原切片的数据，而是将新的切片的指针指向原切片的某个元素。示意如下

![go-slices-usage-and-internals_slice-2](/images/post/go-slices-usage-and-internals_slice-2.png)

```
s[:] == s
as := s[2:4] // 对于上面生命的s
// len(as) = hi - lo = 4-2 =2
// cap(as) == len(s)-lo = 5-2 = 3
```
这样可以理解切片的len和cap的区别了。


可以使用“切片”操作把数组转换为切片

```
arr := [3]int{1,2,3}
sli := arr[:]
```
也就是说可以对数组做“切片”操作。
切片不同于数组的地方在于切片的copy和append方法。

copy方法的签名为:`func copy(dst, src []T) int`; copy方法还可以处理共享相同底层数组的切片，处理dst和src的重叠问题(比如上面的s和as)。当dst和src的长度不一样时，**只会拷贝较短**的部分。理解这一部分这也是很重要的

```golang
sli := []int{1,2,3,4,5}
t := []int{11,21,31,41,5,6,7,8,9}
copy(sli, t) // sli=[11,21,31,41,5]
// copy(t, sli) // t=[1,2,3,4,5,6,7,8,9]
```
append方法用于对切片追加元素。append方法的签名为: `func append(s []T, x ...T) []T`; 方法想s的结尾追加元素x，如果len小于cap则直接追加，如果capacity不够则会自动扩大切片。

append可以用来追加元素也可以用来将一个切片追加到另一个切片后面。这里使用`...`操作符(expand operator)

```
append(t, 100, 200)
append(t, sli...)
```
需要注意的是，append会返回一个新的切片而不是修改原来的切片(试验了一下，append返回的切片不是共享存储)，可以通过append方法的签名看出。

切片共享存储可能存在问题是，如果一个很大的切片“切”出一小片保存在一个新切片，而这个大切片这已经不需要了，但是由于小切片保留了这些引用，所以GC Collector还不能回收这一个大切片。如果真的遇到这样场景推荐罢这小块切片复制出来。

参考:

- [http://blog.golang.org/go-slices-usage-and-internals](https://golang.org/doc/effective_go.html#arrays)
- [https://gobyexample.com/slices](https://gobyexample.com/slices)
- [https://gobyexample.com/slices](https://gobyexample.com/slices)
- [https://golang.org/doc/effective_go.html#arrays](https://golang.org/doc/effective_go.html#arrays)
