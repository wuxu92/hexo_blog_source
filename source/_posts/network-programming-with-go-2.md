---
date: 2015/11/02 12:34:50
title: Go 网络编程-Socket接口与IP套接字
categories:
- programming
tags:
- network
- golang
- go
---
![](/images/golang/gopher-banner-small.jpg)

## UDP数据报 ##
关于UDP的部分在 [这篇](http://wuxu92.github.io/go-lang-notebook-2/) 文章里已经介绍了很多了，这里只记录在npwg中出现的内容。

## 监听多个socket ##
一个server一般式监听多个端口服务多个客户端的，这种情况一般会用到轮训的方式。在C中使用 select() 调用让内核去做这个工作。在Go中，为每一个端口分配一个Goroutine来实现。

## Conn, PacketConn, Listener 类型 ##
前面的TCP和UDP使用了不同的API，比如TCPConn和UDPConn其实它们都是实现了 Conn 接口。所以之前的写法也都可以直接使用 Conn 实现，这样可以不需要在代码中区分 TCP和 UDP而直接使用 `Dial` 方法。

```
func Dial(net, laddr, raddr string) (c Conn, e os.Error)
```
需要注意的是，这里的 laddr 和 raddr 都是字符串，也就是 host:port 这样的形式的字符串，而不需要向之前那样先使用 `net.ResolveUDPAddr` 或者 `net.ResolveTCPAddr` 先解析出地址。因为可以兼顾TCP/UDP/IP数据，所以net参数可以为 `tcp, tcp4, tcp6, udp, udp4, udp6, ip, ip4, ip6`。这个方法返回一个Conn接口的实例。

```
conn, _ := net.Dial("tcp", "zhihu.com:80")
conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n"))
var buffer = make([]byte, 512)
if n, err := conn.Read(buffer); err == nil {
	fmt.Println("response", n, "\n", string(buffer))
} else {
	fmt.Println("err occur", err)
}
```
一个更加简化的使用接口实现的Head程序。但是这样写有一个问题，如果服务器端返回的内容超多了512个字节，那么buffer不能存储这么多，那么就只会读到前512个字节的内容，后面的被丢弃。我们需要是用一个缓冲区来存储（这部分可以封装为一个方法，读取所有返回）。Read部分改如下：

```
result := bytes.NewBuffer(nil)
var buf [512]byte
for {
	n, err ：= conn.Read(buf[0:])
	result.Write(buf[0:n])
	if err != nil {
		if err == io.EOF {
			break
		}
		fmt.Pritnln("err", err)
	}
}
fmt.Println("res", result.Bytes())
```
而对于server端，可以不需要ListenUDP或者ListenTCP而直接使用 `Listen` 方法，它会返回一个实现了 `Listener` 接口的对象。其对应的方法签名如下：

```
func Listen(net, laddr string) (l Listener, err os.Error)
func (l Listener) Accept() (c Conn, err os.Error)
```
同样需要注意Listen的laddr是一个字符串了，而不是解析过的TCPAddr或者UDPAddr。
对应UDP使用 `ListenPacket(net, laddr string) (c PacketConn, err os.Error)` 方法。

## 使用更原始的IP套接字 ##
这一节我们讨论一下一般情况下不太需要的原始套接字，这可以构建我们自己的IP协议或者使用TCP和UDP之外的协议。
TCP和UDP不是IP层上仅有的协议。现在大概有140种这样的协议（参看 `/etc/protocls`）。
Go可以通过原始套接字使用户使用这些协议进行通信。

我们可以使用原始套接字实现一个icmp客户端程序，也就是ping命令。我们要注意的是ping不是传输层应用，不使用TCP或者UDP也就**没有端口的概念**。

```
func main() {
	addr, _ := net.ResolveIPAddr("ip", "baidu.com")
	conn, err = net.DialIP("ip4:icmp", nil, addr)
	CheckErr(err)

	var msg [512]byte   // icmp 报文
	msg[0] = 8
	msg[1] = 0
	msg[2] = 0
	msg[3] = 0
	msg[4] = 0
	msg[5] =  13
	msg[6] = 0
	msg[7] = 37
	len := 8
	check := checkSum(msg[:len])
	msg[2] = byte(check >> 8)
	msg[3] = byte(check & 255)
	_, err = conn.Write(msg[0:len])
	CheckErr(err)
	fmt.Println("wites", (msg[0:len]))
	n, err := conn.Read(msg[0:])
	CheckErr(err)
	fmt.Println("got ping res", n)
	if msg[5] == 13 {
		fmt.Println("ping identifier matches")
	}
	if msg[7] == 37 {
		fmt.Println("ping sequence matches")
	}
}
// 计算校验和
func checkSum(msg []byte) uint16 {
	sum := 0
	for n := 1; n < len(msg)-1; n += 2 {
		sum += int(msg[n])*256 + int(msg[n+1])
	}
	sum = (sum >> 16) + (sum & 0xffff)
	sum += (sum >> 16)
	var ans uint16 = uint16(^sum)
	return ans
}

func CheckErr(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "err %s", err.Error())
		os.Exit(1)
	}
}

```
这段代码在Windows下使用管理员运行也没有结果（使用非管理员运行会提示权限不足），会在`Read(msg[:])` 那里卡住，但是在Linux运行是可以的，不过后面那端matches的不知道干什么的，读到的返回的数据的内容大概是这样的： `[69 0 0 28 45 250 64 0 64 1 216 33 10 4 16 95 10 4 16 95 8 0 242 255 0 13 0 37]`

下一张 数据序列化
