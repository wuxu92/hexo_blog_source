---
date: 2015/08/25 12:34:50
title: Go by Example(四)： atomic&mutex&stateful goroutine
categories:
- programming
tags:
- golang
- go
---

## 原子性计数/atomic ##
在过线程场景下，进行全局的技术是很麻烦的，在java中我们需要使用加锁去实现一段互斥代码，go提供的sync包中有一些专门用来计数的封装。

```
var counter uint64 = 0;
for ci:=0; ci<50; ci++ {
  go func() {
    for {
      atomic.AddUint64(&counter, 1)
      runtime.Gosched()
    }
  }()
}
time.Sleep(time.Second * 1)
cResult := atomic.LoadUint64(&counter)
fmt.Println("counter result is ", cResult)
```
注意sync包的方法都需要传入指针作为实参。还有goroutine中的`runtime.Gosched()`是明确指定go调度器切换上下文（使得其他goroutine也能运行）；在无线有无限循环的goroutine中，应该在合适的地方添加这一调用。可以在 [stackoverflow](http://stackoverflow.com/questions/13107958/what-exactly-does-runtime-gosched-do "http://stackoverflow.com/questions/13107958/what-exactly-does-runtime-gosched-do") 了解更多关于Gosched()的知识。

## 互斥器/mutex ##
mutex在操作系统中是很重要的一部分，在资源调度/分配中，经常需要使用mutex防止资源同时被多个线程修改导致异常。在go中因为可以有多个goroutine执行，所以也存在同步互斥的问题。

[https://gobyexample.com/mutexes](https://gobyexample.com/mutexes "https://gobyexample.com/mutexes")

*待更新*

## goroutine的状态/stateful goroutine ##
我们可以通过除斥锁来在多个goroutine共享状态，另外也可以通过go语言goroutine和channel的内置特性来实现。这种基于channel的实现方式，和goroutine通过通信来共享内存来确保一块数据只被唯一的一个goroutine所有的思路是一样的。

> This channel-based approach aligns with Go’s ideas of sharing memory by communicating and having each piece of data owned by exactly 1 goroutine

实际上并不是goroutine本身拥有某种状态，我们这里讲述的也是一种编程的模式/模型，通过goroutine和channel的结合实现一种互斥访问的机制。
它的思路是这样的，有两个所有goroutine共享的channel，就叫它们reads 和 writes，共享的内存区域/变量由一个goroutine所有，我们把这个共享的变量叫做state，它是一个map；这个goroutine负责从state里面读取或者写入数据，当reads或者writes channel有新的任务到来时（任务由其他goroutine添加）。
其他的goroutine需要读取一个状态的时候，就往reads channel中传入一个值，然后等待reads操作返回。要写入的话也同理。
这里要设计reads和writes的数据结构了。也就是这两个channel的类型，reads是要读取，需要传入一个索引，假设是int型的索引，读取的结果希望也存在一个channel中来实现同步；writes则需要传入索引，值和写入数据以及完成的channel。设计如下：

```golang
type read struct {
  key int
  req chan int
}
type write struct {
  key int
  value int
  req chan bool
}
```
这里的key和chan的类型是有共享的map决定的，我们假设共享的map是`map[int]int`的如果是其他的map则要相应的修改。

这两个结构就是我们通信用的数据。下面新建两个所有goroutine共享的channel:

```golang
reads := make(chan *read) // 注意是指针类型
writes := make(chan *write)
```
创建管理共享变量的goroutine：

```
// state manage gr
go func() {
  state := make(map[int]int)  // 共享的变量
  for {
    select {
      case r:= <-reads:  // 如果reads有新的任务
        r.req <- state[r.key] // 把值写入channel
      case w:= <-writes: // 如果writes有新的任务
        state[w.key] = w.value // 把值写入共享变量
        w.req <- true // 通知返回
    }
  }
}()
```

然后就可以访问共享变量了，读取示例：

```golang
for ri := 0; ri<20; ri++ {
  go func() {
    for {
      r := &read {  // 注意取地址符
        key: rand.Intn(10),
        req: make(chan int),
      }
      reads <- r
      res := <-r.req
      _ = res // 暂时不用
      atomic.AddInt64(&ops, 1)
    }
  }()
}
```

写入示例：

```golang
for wi:=0; wi<10; wi++ {
  go func() {
    for {
      w := &write{
        key: rand.Intn(10),
        value: rand.Intn(100),
        req: make(chan bool),
      }
      writes <- w
      res := <- w.req
      _ = res // 暂时不用
      atomic.AddInt64(&ops, 1)
    }
  }()
}

time.Sleep(time.Second * 1)
opsFinal := atomic.LoadInt64(&ops)
fmt.Println("total ops:", opsFinal)
```
关于使用：

> It might be useful in certain cases though, for example where you have other channels involved or when managing multiple such mutexes would be error-prone. You should use whichever approach feels most natural, especially with respect to understanding the correctness of your program.

