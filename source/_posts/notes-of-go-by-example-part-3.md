---
date: 2015/08/24 12:34:50
title: Go by Example(三)： Timer & Ticker
categories:
- programming
tags:
- golang
- go
---

## 定时器 Timer ##
在之前的部分中，已经使用过超时与睡眠(sleep)了。但是更一般的，我们可能想要某段代码在规定的时间执行，或者重复执行。Go内置的timer和ticker可以方便的实现这些功能。
timer包含在time包中，使用如下：

```
import "time"
...
// 定义一个timer，超时设定2秒
timer1 := time.NewTimer(time.Second *２）
// 在此等待2s
<- timer1.C  //注意.C
```
上面的代码使用timer在某个代码点阻塞等待2秒。但是，如果我们想要实现的知识阻塞等待一段时间，更应该使用time.Sleep()方法。timer的特性在于它是可以取消/stop的。

```
t2 := time.NewTimer(time.Second * 1)
go func() {
	<- t2.C
	... // other code
}()
stop2 := t2.Stop()
```
实际上在上面的代码中，other code部分**不会被执行**，因为goroutine中的t2在主函数中被stop了， <-t2.C 没有机会到达expire点。

## Tickers ##
timer可以定时执行某段代码，而ticker就像打点器一样循环执行某代码。其作用有点像js中的`setTimeout()`和`setInterval()`
同时可以把ticker看作一个channel，在初始化时定义一个循环时间，每过一段时间往channel里面塞一个数，然后我们循环去取，这样就实现了循环执行代码的功能，看起来比js的`setInterval`要麻烦你一点点。

```
t1 := time.NewTicker(time.Millisecond * 500)
go func() {
	for t := range t1.C {
		fmt.Println("tick tick @ ", t)
	}
}()
time.Sleep(time.Second * 3)
t1.Stop()
fmt.Println("tick tick stop")
```
上面会打印6次“tick tick @***",range ticker.C会返回时间戳。
我们看一下ticker的实现：

```
type Ticker struct {
        C <-chan Time // The channel on which the ticks are delivered.
        // contains filtered or unexported fields
}
```
果然起始ticker就是包含一个chan成员的结构体，这样理解起来就更加方便了。

## worker pool/worker池 ##
worker这个概念在很多地方都有，印象比较深刻的是在异步javascript编程那篇文章里面讲了很多worker的东西。
这里worker pool主要是借助channel来实现使用多个goroutine处理多个work的**编程模型**。可以把一个goroutine看作一个worker。给多个worker传入相同的channel，worker从channel中取任务并处理，主线程往channel中添加任务，这样过个goroutine处理一个任务队列。

```
jobs := make(chan int, 100)
result := make(chan int, 100)
// use five workers
for w:=0; w<5; w++ {
  go func(w int, jobs <-chan int, result chan<- int) {
    for j:= range jobs {
      fmt.Println("worker", w, "processing on", j)    
      time.Sleep(time.Millisecond * time.Duration(rand.Intn(600))) // sleep a random time
      result <- j * 2
    }
  }(w, jobs, result)
}

// insert jobs
for j:=0; j<10; j++ {
  jobs <- j
}

close(jobs)

// can do some others here

// sync here, may wait for works done
for a :=0; a<10; a++ {
  <- result
}
fmt.Println("all work done here")
```

## 限速/rate-limiting ##
关于rate-limiting,可以参考 [维基百科](https://en.wikipedia.org/wiki/Rate_limiting "https://en.wikipedia.org/wiki/Rate_limiting")

> Rate limiting is an important mechanism for controlling resource utilization and maintaining quality of service

go使用goroutine，channel， ticker合作实现该特性。

使用rate-limiting的模型，可以应对突然的大量请求进行缓存，有序处理。

```
rateControlChan := make(chan time.Time, 3) // 用于控制速率的channel
for i:=0; i<3; i++ {
  rateControlChan <- time.Now()
}
// send value per 200 millisecond
go func() {
  for t := range time.Tick(time.Millisecond * 200) {
    rateControlChan <- t
  }
}()

reqChannel := make(chan int, 5) // 缓存请求的channel
for i:=0; i< 5; i++ {
  reqChannel <- i
}
close(reqChannel)
for r:= range reqChannel {
  ht := <- rateControlChan // wait for rate-control condition
  // handle request here
  fmt.Println("req", r, "handle at", ht)
}
```
这个模型的关键是，使用两个channel协同，一个用于处理速率的控制，一个用于请求的缓冲。使用一个goroutine不断的定时填充速率控制goroutine，而请求channel则用一个for-range不停行从里面取请求

## 待续 ##
*下一部分主要是原子性，同步与互斥*
