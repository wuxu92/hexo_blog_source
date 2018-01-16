---
date: 2015/10/29 12:34:50
title: Go 网络编程
categories:
- programming
tags:
- network
- golang
- go
---
![](/images/golang/gopher-banner-small.jpg)

对于任何一门语言，网络编程都是重要的一部分。对于Go来说其天生的高并发网络编程更是充满魅力。所以今天开学学习Go网络编程部分，教材是 Jan Newmarch 的 Network programming with Go 的pdf文档。 这份文档在 [这里](https://jan.newmarch.name/go/) 可以找到，由 Astaxie 翻译的中文版也在[这个网站的这个目录](https://jan.newmarch.name/go/zh/index.html)， 当然也可以在[astaxie的github上](https://github.com/astaxie/NPWG_zh)找到。

我使用英文的pdf，因为没有找到中午的pdf，而使用pdf比网页要方便些，方便做笔记和添加备注，并且这类文档的英文应该也不会太难。

根据pdf的结构，一节一节往后走。各章节组成如下：

1. 架构
2. Go语言概括
3. Socket编程
4. 数据序列化
5. 应用层协议
6. 管理字符集与编码
7. 安全
8. HTTP
9. 模板
10. 一个完整的Web服务
11. HTML
12. XML
13. 远程过程调用RPC
14. 网络 Channels
15. Web Sockets

其中前两张只简单的过一下，重要的是这几节： Socket, 序列化，应用层协议，安全，HTTP。 RPC与Web Sockets。其他章节看起来简单过一下就可以了。
<!-- more -->
## 基础/架构 ##
不同领域的程序员会有不同的技能树～批处理程序，Gui开发，游戏开发，分布式系统开发等不同的领域必然各不相同的。干什么活呢需要不同的工具，有各自的方法和模式。我们现在要考虑的就是高层的分布式系统的架构。

分布式系统的相关概念已经比较熟悉了，但是还没有真正的开发过一个分布式系统。因为光知道原理就觉得这个东西很难。网络通信需要协议来为高层应用提供支持，关于ISO/OSI协议栈也比较熟悉了就不多说了。现在用的最多的是TCP/IP协议栈，比OSI简单很多，它只有4层。(注意计算机网络中学的5层协议是OSI和TCP/IP为了教学混合出来的)。分别是： 硬件层，网络层，传输层，应用层。

**面向连接和无连接**。这是两个很重要的概念。面向连接是在会话时会建立一条连接，交互的通信在这个连接上进行。当会话结束是连接断开，比如TCP；而无连接的通信是每一个消息都是独立发送的，就像发邮件一样，比如IP协议，通常面向连接的服务是基于无连接服务。要注意区分面向连接和持久连接。面向连接是通信中复用一条链路，而持久连接是多个会话公用一次连接，参考http的keep-alive。

### 消息传递 ###
在网络服务中消息传递是很重要很基础的部分。其实Unix 管道通信就可以看作是一类消息传递，它直接传输二进制流。消息传递是很基础的功能，一般通过消息传递来共享。一个很重要的准则就是

> 不要通过共享来通信，而是通过通信来共享


![](/images/golang/messages.png)
消息传递用在大部分cs架构中，X window system(Linux的图形界面服务)就是这样的机制。server等待用户发送的消息（鼠标键盘信息），然后处理消息，返回结果。

- 远程过程调用(RPC)
- 分布计算模型： 点对点，过滤器，CS


## Socket编程 ##
TCP/IP协议栈的东西就不记录了。

### net包中定义的IP地址 ###
net 包是网络编程中最重要的包，它定义了很多相关的类型，函数，方法。`IP`是其中最重要的一个类型，它的定义如下：

```golang
type IP []byte
```
IP的基础类型是 `[]byte` 即byte的切片，我们知道IPv4的地址是4-byte（32bit）， IPv6地址是16-byte。IP使用切片作为基础类型可以同时支持IPv4和IPv6。但是不要通过切片的长度来判断IP的类型，因为16-byte的切片也可能保存的是IPv4地址。
net包提供了很多的方法调用。比如将string类型转换为IP类型的`net.ParseIP(string)`

**IPmask**: 掩码相关 `type IPmask []byte`, 一个IP mask 也会死一个ip地址。子网掩码有几个很重要的函数。

```
// 通过 4-byte 生成一个IPMask变量
func IPv4Mask(a, b, c, d byte) IPMask

// 获取IP的默认子网掩码
func (ip IP) DefaultMask() IPMask

// 获取IP地址的网络地址
func (ip IP) Mask(mask IPMask) IP
```
**IPAddr** net包中很多函数都返回一个IPAddr指针。IPAddr的定义很简单的就是包含了一个IP。 

```
type IPAddr struct {
	IP IP
}

// 使用,ResolveIPAddr(string, string) 返回域名对应的IPAddress
addr, _ := net.ResolveIPAddr("ip", "baidu.com")
	if addr == nil {
		fmt.Println("Invalid address")
	} else {
		fmt.Println("the address is ", addr.String())
	}
```
net包还有一个LookupHost的函数，可以用来查找CNAME(canonical name)

**服务/service** 服务是持续运行在服务器端等待客户端请求的程序，服务有很多中，但是internet中大部分服务都是基于UDP和TCP通信，诸如SCTP这些协议还没有大量使用。

对计算机网络有一定了解的话，我们知道IP定位到服务器，但是一个机器可以提供很多的服务，这时使用port来区分不同的服务。一个服务可以监听一个或者多个端口，端口分为TCP，UDP端口。一些常用的默认端口如下：

- Telnet 23 TCP
- DNS 53 TCP/UDP(or)
- 21/20 FTP
- 22 SSH TCP
- X window system 6000-6007  TCP/UDP(both)

更详细的端口与服务对应列表可以在linux系统的 `/etc/services` 文件中找到。Go中提供了一个 `func LookupPort(network, service string) (port int, err os.Error) ` 方法用于查询服务对应的端口。network参数是 tcp 或者 udp。

```
port, err := net.LookupPort("tcp", "ftp")
if err != nil {
	fmt.Println("lookupPort err:", err)
} else {
	fmt.Println("port for svn on tcp", port)
}
```
**TCPAddr** TCP地址，是IP加上端口的类型。

```
type TCPAddr struct {
	IP IPp
	Port int
}
```
创建一个TCPAddr的方法是 `ResolveTCPAddr(net, addr string) (*TCPAddr, os.Error)` 其中的net是网络类型，如 tcp, tcp4, tcp6

```golang
ta, err := net.ResolveTCPAddr("tcp6", "localhost:6060")
if err != nil {
	fmt.Println("tcpaddr err:", err)
} else {
	fmt.Println("tcp addr:", ta)   // [::1]:6060
}
```

### TCP Sockets ###
如果我们要建立一个服务器，需要绑定一个端口并且监听之。当有消息请求到这个端口时要能够读取请求的内容并把处理的结果返回给请求者。

Go提供的 `net.TCPConn` 类型提供了client和server之间的全双工通信,它能够同时用作client和server，提供的最重要的两个函数如下：

```
func (c *TCPConn) Write(b []byte) (n int, err os.Error)
func (c *TCPConn) Read(b []byte) (n int, err os.Error)
```
**TCP Client** 要获得TCPConn的实例，使用 `net.DialTCP(net string, laddr, raddr *TCPAddr) (c *TCPConn, err os.Error)`；其中net同上表示tcp, tcp4, tcp6等, laddr表示localAddr，一般使用nil，raddr表示要请求的Remote Address。

使用上面的内容对网站发送 HEAD 请求：

```
import (
	"fmt"
	"os"
	"io/ioutil"
	"net"
)

func main() {
	addr, e := net.ResolveTCPAddr("tcp4", "baidu.com:80")
	CheckErr(e)
	conn, e := net.DialTCP("tcp", nil, addr)
	CheckErr(e)
	_, e = conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n"))
	CheckErr(e)
	res, e := ioutil.ReadAll(conn)
	CheckErr(e)
	fmt.Println(string(res))
}

func CheckErr(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "error:", err)
		os.Exit(1)
	}
}
```

返回信息如下：

```
HTTP/1.1 200 OK
Date: Mon, 02 Nov 2015 07:02:02 GMT
Server: Apache
***
```
HEAD 请求有时候很有用，比如用来查看网络是否可达，它只发送很少的数据且服务器只返回连接信息。

我们的代码里出现了很多的错误检查，这是不得已的，因为网络编程很多地方都容易出错，比如硬件错误，路由错误，防火墙阻止，超时等等。由于缺少异常系统的 try-catch 只能分别对每个错误进行检查。
我们知道TCP的数据是按包发送的，所以实际读的是一个数据流，不过ioutil.ReadAll 方法会处理这个问题，它会一直等待读取直到所有包都收到。

**TCP server** 搭建一个TCP server也很简单，主要使用的方法是 `func ListenTCP(net string, laddr *TCPAddr) (l *TCPListener, err os.Error)` 和 `func (l *TCPListener) Acccept() (c Conn, err os.Error)` 两个方法。

下面是一个简单的TCP server 的搭建示例：

```
import (
	"fmt"
	"net"
	"os"
	"time"
)

func main() {
	port := ":1200"
	tcpAddr, err := net.ResolveTCPAddr("tcp4", port)
	CheckErr(err)
	server, err := net.ListenTCP("tcp", tcpAddr)
	CheckErr(err)
	for {
		conn, err := server.Accept()
		if err != nil {
			continue
		}
		time := time.Now().String()
		conn.Write([]byte(time))
		conn.Close()
	}
}
```
其中的CheckErr和之前的示例是一样的。这个server会对任意的请求返回当前的服务器时间。可以使用telnet工具连接 `telnet localhost 1200` 会返回一个时间，并被关闭连接。
当然我们可以使用Goroutine执行各个连接的处理。这个之前用的挺多的了，不详细写了。

**Timeout和Keep-alive** 我们知道浏览器的请求有超时和keep-alive的选项，对于TCP连接也可以设置这两个参数。

```
func (c *TCPConn) SetTimeOut(nsec int64) os.Error
func (c (TCPConn) SetKeepAlive(keepalive bool) os.Error
```

### UDP数据报 ###
见下一篇
