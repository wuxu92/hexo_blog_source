---
date: 2015/07/20 12:34:50
title: Effective Go笔记：并发
categories:
- programming
tags:
- golang
- lang
---

effective go 中的部分笔记，这是第四部分 

部分笔记摘要,参考： [https://go-zh.org/doc/effective_go.html](https://go-zh.org/doc/effective_go.html "https://go-zh.org/doc/effective_go.html")

## 并发 ##

### 通过通信共享内存 ###
并发编程中很麻烦的一点是对共享变量访问的控制，在一般环境中，使用同步互斥锁机制等实现互斥的访问，使得代码繁琐而且难以理解。Go语言提出了一个独特的机制:信道。
Go将共享的值通过信道传递。在给定的时间点，只有一个Go程能访问信道中的值，在设计上杜绝了数据在同一时间点被放线程访问。

> 不要通过共享内存来通信，而应通过通信来共享内存。

为了方便理解，可以把Go的并发处理方式看做类型安全的Unix管道的实现。

### Go程/Goroutines ###
Go程是Golang提出的术语，Go程有简单的模型，它是与其他Go程并发运行在同一地址空间的**函数**。Go程可以看作是轻量级的线程，它的开销几乎只有栈空间的分配，它的使用很廉价。
Go程在多线程操作系统上可以实现多路复用，后面会详细讲到。多路复用在多线程编程中非常重要。

那如何运行一个Go程呢？很简单，只需要在要调用的函数前添加"go"关键字就能在Go程中运行这个函数，函数调用结束时，Go程也就自动退出。

```golang
go list.Sort()
// 使用函数字面量运行
go func() {
	// do something here
}
```

### 信道/Channels ###
前面已经提到了信道的概念，在多线程中，具体怎么使用呢，下面就详细介绍**信道（channel）**。

之前讲过，在Go中切片，映射和信道都使用make来分配内存，make返回的值充当对底层数据结构的引用。信道的声明和初始化示例如下：

```
// 前两个是无缓冲的正数类型信道
c1 := make(chan int)
c2 := make(chan int, 0)
// 指向文件指针的带缓冲区的信道
c3 := make(chan *os.File, 100)
```
若不指定第二个参数，信道就是无缓冲的同步信道。使用一个同步信道，一个示例如下：

```golang
c1 := make(chan int) // 一个不带缓冲的，同步信道
go func() {
  time.Sleep(500 * time.Millisecond)
  // send one value to channl
  c1 <- 1
}()
// do something here
fmt.Println("wait for goroutine to quir")
<-c1 // 主线程在此等待Go程结束,丢弃信道中的值，因为该值只是用作同步的标识
fmt.Println("go routine done")
```
看完无缓冲的信道，下面看一下有缓冲的信道。有缓冲的信道可以看作是信号量，操作系统里面学过信号量的东西。信号量可以看作是资源不足时用来限制线程执行，使其等待的机制。常用来限制吞吐量，限制调用的process的数量等。
比如要对Web请求使用Go程处理：

```
func Serve(queue chan *Request) {
	for req := range queue {
		sem <- 1
		// 考虑到闭包的性质，这里传入值 *Request，
		// 如果不使用 *Request则所有Go程处理的是同一个变量req
		go func(req *Request) {
			process(req)
			<-sem
		}(req)
	}

	// 另一种方式是，重新声明一个变量
	for req := range queue {
		// 参考js中for循环中使用timeInterval函数时要重新声明变量
		req := req // 为该Go程创建 req 的新实例。
		sem <- 1
		go func() {
			process(req)
			<-sem
		}()
	}
}
```

### 信道中的信道/channels of channels ###
*这部分比较麻烦，待续*

### 并行化/Parallelization ###
在多CPU核心上实现并行计算。

> 并发是用可独立执行的组件构造程序的方法， 而并行则是为了效率在多CPU上平行地进行计算。Go仍然是种并发而非并行的语言，且Go的模型并不适合所有的并行问题

目前Go运行时的实现默认并不会并行执行代码，它只为用户层代码提供单一的处理核心。 任意数量的Go程都可能在系统调用中被阻塞，而在任意时刻默认只有一个会执行用户层代码。 它应当变得更智能，而且它将来肯定会变得更智能。但现在，若你希望CPU并行执行， 就必须告诉运行时你希望同时有多少Go程能执行代码。有两种途径可以使用：

1. 在运行你的工作时将 GOMAXPROCS 环境变量设为你要使用的核心数
2. 导入 runtime 包并调用 runtime.GOMAXPROCS(NCPU)。 runtime.NumCPU() 的值可能很有用，它会返回当前机器的逻辑CPU核心数。 

当然，随着调度算法和运行时的改进，将来会不再需要这种方法。

### 一个缓冲区泄漏示例/A leak buffer ###
客户端Go程从某些来源，可能是网络中循环接收数据。为避免分配和释放缓冲区， 它保存了一个空闲链表，使用一个带缓冲信道表示。若信道为空，就会分配新的缓冲区。 一旦消息缓冲区就绪，它将通过 serverChan 被发送到服务器。 serverChan.

```golang
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
	for {
		var b *Buffer
		// 若缓冲区可用就用它，不可用就分配个新的。
		select {
		case b = <-freeList:
			// 获取一个，不做别的。
		default:
			// 非空闲，因此分配一个新的。
			b = new(Buffer)
		}
		load(b)              // 从网络中读取下一条消息。
		serverChan <- b   // 发送至服务器。
	}
}
```
服务器从客户端循环接收每个消息，处理它们，并将缓冲区返回给空闲列表。

```golang
func server() {
	for {
		b := <-serverChan    // 等待工作。
		process(b)
		// 若缓冲区有空间就重用它。
		select {
		case freeList <- b:
			// 将缓冲区放大空闲列表中，不做别的。
		default:
			// 空闲列表已满，保持就好。
		}
	}
}
```
客户端试图从 freeList 中获取缓冲区；若没有缓冲区可用， 它就将分配一个新的。服务器将 b 放回空闲列表 freeList 中直到列表已满，此时缓冲区将被丢弃，并被垃圾回收器回收。（select 语句中的 default 子句在没有条件符合时执行，这也就意味着 selects 永远不会被阻塞。）依靠带缓冲的信道和垃圾回收器的记录， 我们仅用短短几行代码就构建了一个可能导致缓冲区槽位泄露的空闲列表。(没懂TT)

上一段的英文如下，总觉得中文的翻译不太对

The client attempts to retrieve a buffer from freeList; if none is available, it allocates a fresh one. The server's send to freeList puts b back on the free list unless the list is full, in which case the buffer is dropped on the floor to be reclaimed by the garbage collector. (The default clauses in the select statements execute when no other case is ready, meaning that the selects never block.) This implementation builds a leaky bucket free list in just a few lines, relying on the buffered channel and the garbage collector for bookkeeping.


