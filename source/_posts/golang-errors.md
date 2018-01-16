---
date: 2015/07/24 12:34:50
title: Golang中的错误处理
categories:
- programming
tags:
- golang
- lang
- exception
---

Go语言提供了一套“新颖的”错误系统，与Java中的异常系统不一样，Go中没有抛出异常的概念

## 基础知识 ##
如果有一定面向对象语言编程经验的话，肯定会对异常/错误系统有一定的了解。如果使用Java比较多的话，一定会认为抛异常是编程语言天生的一部分，因为在Java中抛异常真的是太家常便饭，无处不在了；但是在Go语言中，根本没有异常这个东西！

看一段官方的QA是怎么解释这个问题的：

> We believe that coupling exceptions to a control structure, as in the try-catch-finally idiom,** results in convoluted code**. It also tends to encourage programmers to label too many ordinary errors, such as failing to open a file, as exceptional.
>
> Go takes a different approach. For plain error handling, Go's multi-value returns make it easy to report an error without overloading the return value. A canonical error type, coupled with Go's other features, makes error handling pleasant but quite different from that in other languages.
>
> Go also has a couple of built-in functions to signal and recover from truly exceptional conditions. The recovery mechanism is executed only as part of a function's state being torn down after an error, which is sufficient to handle catastrophe but requires no extra control structures and, when used well, can result in clean error-handling code.

多值返回为Go提供了一个新的错误处理机制，如果对为什么Go不提供异常体系感兴趣可以在 [这里](https://golang.org/doc/articles/defer_panic_recover.html) 找到更多的细节。

在之前文章中，已经使用过Go语言的多值返回来返回详细的错误描述。按照规定，**错误的类型通常为error，这是一个内建的简单接口**。

```golang
type error interface {
	Error() string
}
```

可以通过实现这个接口定制错误，例如打开文件时的错误:

```golang
type PathError struct {
	Op string
	Path string
	Err error	// 由系统调用返回
}

fanc (pe *PathError) Error() string {
	// 返回定制的错误信息
	return pe.Op + " " + pe.Path + ": " + pe.Err.Error()
}
```
effective go中有一个示例非常有利于我们理解错误的使用,这段代码的作用是创建一个文件，如果创建失败，检查是不是磁盘空间不足，如果是则删除掉一些临时文件，再尝试创建一次：

```
for try:=0; try<2; try++ {
	file, err := os.Create(filename)
	if err == nill {
		return  //没有错误
	}
	// 检查是不是空间不足
	if e,ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
		deleteTempFiles()  	// 删除一些临时文件
		continue 			// 再创建一次
	}
	return
}
```
可以查看源码中PathError的实现 [https://golang.org/src/os/error.go](https://golang.org/src/os/error.go "https://golang.org/src/os/error.go")

## Panic函数 ##
函数可以通过将error作为额外的返回值来向调用者报告错误，对于一些“致命”的错误，他们导致程序不能继续运行了，Go内建了一个Panic函数，它会产生一个运行时错误并重孩子程序。该函数接受一个任意类型的实参在程序终止时打印。

只需要在需要它的地方调用一下`panix()`就可以了。

```
func init() {
	if user == "" {
		panic("user变量必须有值")
	}
}
```

这是一个示例，但是如果是设计库函数则应该避免使用panic，一个常见的使用panic的场景是初始化，如上面的示例。

## 恢复 ##
panic函数被调用后，程序将终止当前函数的执行。并开始追溯Go程的栈，运行任何被推迟的函数。我如果回溯到栈顶，程序会终止。
我们可以通过使用内建的recover函数来重新取回Go程的控制权。
调用recover将停止回溯，并返回传入panic的是西安。因为回溯只有被推迟执行的函数中的代码在执行，所以recover也只能在被推迟的函数中才有效。

一个应用：在服务器中终止失败的Go程而无需杀死其他正在执行的Go程：

```golang
func server(workChan <-chan *Work) {
	for work := range workChan {
		go safelyDo(work)
	}
}

func safelyDo(work *Work) {
	// defer 函数是被推迟执行的
	defer func() {
		if err := recover(); err != nil {
			log.Println("work failed:", err)
		}
	}()
	do(work)
}
```
如果do函数中调用panic函数，回溯会执行defer的函数，这时recover会起作用，打印错误信息后退出Go程。
可以使用上面的思路简化错误处理。
