---
date: 2015/08/23 12:34:50
title: Go by Example(二)： goroutine & channel
categories:
- programming
tags:
- golang
- go
---

## Goroutines ##
goroutines就是可以方便的并行执行函数，本身并没有太多东西要注意的，主要的内容在于goroutine和channel结合的并发程序

## Channel ##
channel相当于联系并发的goroutine的管道，我们常使用Liux中的管道pipe做类比：

```bash
ls -l | grep .go
```
上面的命令把当前目录下的文件列出，每个文件占一行，用管道把输出结果送到grep程序的标准输入流，grep把当前目录下的go源程序文件筛选出来。管道吧ls程序和grep程序连接起来，goroutine的作用也差不多，把一个goroutine的输出，可以在另外一个goroutine读出来。
当然channel要比Liux的管道功能更加强大一些，因为是可编程的，它可以传递任意规定的类型的变量，使用也很方便。

```golang
msgChannel := make(chan string)

go func() {msgChannel <- "hello"}()
msg := <-msgChannel // receive a msg from channel
```
上面的代码即声明了一个可传递string类型的channel。第3行代码则使用goroutine执行一个匿名函数，函数的作用是向msgChannel传入一个字符串。

> goroutine执行一个匿名还是得语法是很有用的，其格式为`go func() { ... }()`注意最后的括号表示执行匿名函数。

`channel <-`通常叫做发送操作，即向channel发送一个值。` <- channel`通常叫做接受操作，即从channel中取出一个值。
channel的发送和接受是一个同步的过程，只有sender和receiver都准备就绪的时候才能发送/接受，否则会阻塞等待，这在后面的Channel Buffering 中会讲到。

### Channel Buffering ###
默认的channel是不带缓冲的（unbuffered）如果像上面的方式定义channel的话，只能往里面添加一个元素，并且需要有从channel取数据的receiver，否则再往里面添加元素则会导致无限等待。
所以更常见的是使用带缓冲的channel，这样可以一次性往channel里面添加多个元素。

```golang
msgChannel := make(chan string, 2)

msgChanenl <- "hello"
msgChannel <- "world"

fmt.Println(<-msgChannel)
fmt.Println(<-msgChannel)
```
这样就channel就可以缓存两个变量了。在实际使用中，常会使用更大的缓存channel。

### 使用channel同步goroutine ###
我们知道channel的发送和接受操作只有在channel准备就绪时才会进行，否则会阻塞等待，也就是说在向channel发送数据时，需要channel有空余空间，从channel接受数据时，需要channel不能为空，否则也会阻塞等待。如下的代码展示了一个使用不带缓冲的channel实现同步的功能:

```golang
func working(flagChan chan bool) {
  fmt.Println("working on it")
  time.Sleep(2 * time.Second)
  fmt.Println("work done")            

  flagChan <- true
}

func chanSync() {
  flagChan := make(chan bool, 1)
  go working(flagChan)

  // 下面的接受操作会阻塞等待goroutine执行结束
  done := <- flagChan 
  fmt.Println("sync goroutine output", done)
}
```

### 指定channel的方向（操作） ###
我们可以在方法的定义的参数列表处指定channel参数的操作方式为发送或者接受，因为一般发送是一个逻辑，接受是一个逻辑，通常不会把两个操作放在一起，指定某个函数的操作模式可以起到“代码即文档”的作用。比如上面的working函数：

```go
func working (flagChan chan<- bool) { ... }

func recv (flagChan <-chan bool) { ... }
```
即表明了在函数中channel的数据流向（directions）
网页中的代码示例：

```
package main
import "fmt"
func pong(pings <-chan string, pongs chan<- string) {
    msg := <-pings
    pongs <- msg
}
func ping(pings chan<- string, msg string) {
    pings <- msg
}
func main() {
    pings := make(chan string, 1)
    pongs := make(chan string, 1)
    ping(pings, "passed message")
    pong(pings, pongs)
    fmt.Println(<-pongs)
}
```

## 多路goroutine: select ##
select用于从多个channel中轮询状态，从已经准备就绪的channel中接受值并执行相应的代码。select的语法和switch有点相似。请看代码：

```
import (
	"math/rand"
	"time"
	"strconv"
)

func gr1(c chan string) {
  dur := rand.Intn(300)
  time.Sleep(time.Millisecond * time.Duration(dur))
  c <- "gr1 sleep done " + strconv.Itoa(dur)
}

func gr2(c chan string) {
  dur := rand.Intn(400)
  time.Sleep(time.Millisecond * time.Duration(dur))
  c <- "gr2 sleep done " + strconv.Itoa(dur)
}

func selects() {
  c1 := make(chan string)
  c2 := make(chan string)
  go gr1(c1)
  go gr1(c1)
  go gr2(c2)
  go gr2(c2)

  for i:=0; i<4; i++ {
    select {
    case msg := <-c1:
      fmt.Println(msg)
    case msg2 := <-c2:
      fmt.Println(msg2)
    }
  }
}
```
要注意，channel的send与receive操作的个数必须是匹配的，否则会无限阻塞等待。

## 超时 ##
在访问资源的代码中，我们常需要绑定一个超时时间，如果超过一段时间没有加载到资源，或者某个调用没有返回则需要做出相应的处理。
一个例子就是在channel的接受操作，等待一定时间没有资源可以接受则执行预定代码。可以在select中添加一个超时：

```
select {
case res <- c1 :
	fmt.Println(res)
case <-time.After(time.Seconde * 2)
	fmt.Println("timeout")
}
```
可以理解，time.After()是一个特殊的channel。

## 非阻塞的channel操作 ##
在上面的例子里面，channel进行send/receive操作时，是阻塞的，有什么办法让channel的操作变得不是阻塞的呢，比如从一个channel中取数据，如果没有数据则直接执执行一个默认的操作，也不要想刚刚讨论的超时那样设置要给time.After().

刚刚我们说了select的语法和switch很相似，那switch中有default可以用来指定当所有case不没有match到的情况，select是不是也有呢？答案是肯定的。

我们可以为select操作添加一个default分支。一般在所有case之后。这样在所有case的channel都没有就绪的时候执行default的分支代码。

```
select {
case msg := <-c1:
  fmt.Println(msg)
case msg2 := <-c2:
  fmt.Println(msg2)
default:
  // non-blocking channel
  fmt.Println("select default")
}
```

## 关闭channel ##
关闭一个channel意味着不再有数据发送到channel中，这在给接收者一个通信完成的信号时是很有用的。
关闭channel的操作很简单，直接调用`close(chan)`方法即可（在所有数据被发送完后调用close）。
在channel的接受操作时，其实可以指定两个变量:

```
msg, more := <- msgChannel
```
如果channel被关闭了，则more被赋值为false，否则more的值为true。 可以用`_, more := <- channel` 判断一个channel是否被关闭了。

## range over channels ##
可以使用range遍历一个channel中的元素。使用方法也很简单

```
func rangeChannel() {
  c := make(chan string, 2)
  c <- "tomorrow is monday"
  c <- "fee out"
  close(c)
  for str := range c {
    fmt.Println(str)
  }
}
```
要注意，对于一个close了得channel，range能自动判断是否遍历结束，但是如果channel没有调用过close，在上面的代码中，会在接受第三个值的时候阻塞(异常退出)。

## 待续 ##
*第二部分结束* 

last update: 2015-08-23 10:38:00
