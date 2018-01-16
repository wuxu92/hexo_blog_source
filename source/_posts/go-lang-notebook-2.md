---
date: 2015/10/23 12:34:50
title: Golang 学习笔记：HTTP, TCP/IP, UDP
categories:
- programming
tags:
- go
- golang
- lang
---

## HTTP ##
我们知道，go内置有http server 支持，我们只需要在代码中启动这个server就可以启动一个类似于apache的服务器。同时可以很方便的监听多个端口等等。

```golang
import (
	. "fmt"
	"net/http"
)

const (
	PORT = ":1024"
	MSG = "hello, gopher"
)
func main() {
	http.HandleFunc("/hello", Hello)
	go func() {
		http.ListenAndServe(PORT, nil)
	}()
	
	go func() {
		http.ListenAndServeTLS(":443","cert.pem", "key.pem", nil)
	}()
	for {}
}
func Hello(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "text/plain")
	Fprintf(w, MSG)
}
```
这段代码启动了一个server，开启了http和https支持；分别监听在1024端口和443端口。
有几个需要注意的地方。
<!-- more -->
1. http.HandleFunc(string, func(http.ResponseWriter, *http.Request)) 绑定某个请求路径的处理方法
2. 使用goroutine执行监听与服务动作。启动了两个goroutine
3. 最后使用一个for循环保证主线程不会退出。因为主线程退出的话goroutine也会结束。
4. 其中的TLS支持的HTTPS服务的cert和key文件可以使用Go源码中带的文件生成`go run $GOROOT/src/crypto/tls/generate_cert.go -ca=true -host="localhost"
`

最后一个无限循环可以使用channel代替，可以使用sync.WaitGroup实现， sync.WaitGroup持有一个计数器，其`Add(int)`方法增加计数器，其`Done()`方法减小计数器，其`Wait()`方法会让线程阻塞等待计数器恢复0。其实现如下。

```golang
import (
	. "fmt"
	"net/http"
	"sync"
)

const (
	PORT = ":1024"
	MSG = "hello, gopher"
)
var servers sync.WaitGroup

func main() {
	http.HandleFunc("/hello", Hello)
	Launch(func() {
		http.ListenAndServe(PORT, nil)
	})
	
	Launch(func() {
		http.ListenAndServeTLS(":443","cert.pem", "key.pem", nil)
	})
	
	servers.Wait()
}

func Launch(f func()) {
	servers.Add(1)
	go func() {
		defer servers.Done()
		f()
	}()
}

func Hello(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "text/plain")
	Fprintf(w, MSG)
}
```
使用上面的方法可以添加任意多的监听端口。可以通过Server做更多的配置。

```golang
s := &http.Server{
	Addr:           ":8080",
	Handler:        myHandler,
	ReadTimeout:    10 * time.Second,
	WriteTimeout:   10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
log.Fatal(s.ListenAndServe())
```

### HTTP与Go程结合 ###
我们对Goroutine和HTTP都比较熟悉了，我们可以设置一个任务队列，让每一个请求都进入这个队列然后使用一定数量的Goroutine去执行逻辑。这样效率比上面的实现要高很多。

### 发送GET/POST请求 ###
http保重提供了非常方便的发发送get请求和post请求的封装。

```golang
resp, err := http.Get("http://example.com/")
...
resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)
...
resp, err := http.PostForm("http://example.com/form",
	url.Values{"key": {"Value"}, "id": {"123"}})

defer resp.Body.Close()  // 不要忘记defer关闭资源
body, err := ioutil.ReadAll(resp.Body)  // 读取资源
strCont := string(body)
```

## 环境变量 ##
使用os包中的`Getenv(string)`和`Setenv(string, string)`方法获得和设置环境变量。要注意Setenv的作用域只是当前运行进程，结束后或者重新运行后环境变量会重置为系统设置值。

```golang
Println(os.Getenv("PATH"))
os.Setenv("GOOO", "gofast")
os.Setenv("PATH", ".")
Println(os.Getenv("PATH"))
```

## 操作系统信号 ##
当程序在命令行中运行的时候，一般使用`Ctrl+c`可以终结程序运行。对于操作系统有一定了解的话应该知道，`Ctrl+c`向运行的进程发送了一个结束进程的signal。这种方式很有用，但是有点时候暴力结束进程执行不是我们想要的逻辑，我们可以捕获这个**termination signal**，然后对他做一些自己的处理。注意，windows下不能捕获这些信号，只在*nix下有效。

使用os/signal包提供的封装。

```golang
package main

import (
	"fmt"
	"os"
	"os/signal"
	"time"
)
var i int

func init() {
	go SignalHandler(make(chan os.Signal, 1))
}
func main() {
	i = 0
	for {
		i++
		time.Sleep(time.Second)
	}
}

func SignalHandler(c chan os.Signal) {
	signal.Notify(c, os.Interrupt)
	for s:= <-c;; s= <-c {
		switch s {
			case os.Interrupt:
				fmt.Println("interrupt received", i)
				os.Exit(0)  // 去掉这一行，那么Ctrl+c的快捷键将不能中断进程运行
			default:
				fmt.Println("default case")
		}
	}
}
```
运行上面的代码，在Ctrl-c时可以看到输出 interrupt received 字符串。

除了接收系统的信号，还可以向其他的进行发送信号。

```golang
syscall.Kill(syscall.Getpid(), syscall.SIGABRT)
```
这一部分可能暂时用不上，只要知道有这种用法就可以了。

## TCP/IP ##
使用net提供的TCP/IP封装可以很方便的监听与请求端口。

server(src/wuxu/tcpserver/main.go):

```golang
package main

import (
	"net"
	"fmt"
)
var i int
func main() {
	i = 0
	if listener, e := net.Listen("tcp", ":1024"); e == nil {
		for {
			if conn, e := listener.Accept(); e==nil {
				go func(c net.Conn) {
					defer c.Close()
					fmt.Fprintln(c, "hihi", i)
					i++
				}(conn)
			}
		}
	} else {
		fmt.Println("error")
	}
}
```

client(src/wuxu/tcpclient/main.go)

```golang
package main

import (
	"fmt"
	"net"
	"bufio"
)

func main() {
	if conn, e := net.Dial("tcp", "localhost:1024"); e == nil {
		defer conn.Close()
		if text, e := bufio.NewReader(conn).ReadString('\n'); e ==nil {
			fmt.Println(text)
		}
	} else {
		fmt.Println("error", e)
	}
}
```

### 加密的TCP通信 ###
还可以很方便地简历加密的连接，为了加密连接我们需要两套密钥对，即服务器的公钥与私钥，客户端的公钥与私钥，使用之前的方法再在服务器和客户端的目录下，各生成一对密钥。`go run $GOROOT/src/crypto/tls/generate_cert.go -ca=true -host="localhost"`
修改代码如下：

server:

```golang
package main

import (
	"fmt"
	"crypto/tls"
	"crypto/rand"
)
var i int
func main() {
	i = 0
	var config tls.Config
	if certificate, e := tls.LoadX509KeyPair("cert.pem", "key.pem"); e ==nil {
		config = tls.Config{
			Certificates: []tls.Certificate{certificate},
			Rand: rand.Reader,
		}
	}
	if listener, e := tls.Listen("tcp", ":1024", &config); e == nil {
		for {
			if conn, e := listener.Accept(); e==nil {
				go func(c *tls.Conn) {
					defer c.Close()
					fmt.Fprintln(c, "secure hihi", i)
					i++
				}(conn.(*tls.Conn))
			}
		}
	} else {
		fmt.Println("error")
	}
}
```
可以看出，引入tls包后不需要net包了，tls实现了相关得了协议的加密版本。

client:

```golang
package main

import (
	"fmt"
	"bufio"
	"crypto/tls"
)

func main() {
	var config tls.Config
	if certificate, e := tls.LoadX509KeyPair("cert.pem", "key.pem"); e == nil {
		config = tls.Config{
			Certificates: []tls.Certificate{certificate},
			InsecureSkipVerify: true,
		}
	}
	if conn, e := tls.Dial("tcp", "localhost:1024", &config); e == nil {
		defer conn.Close()
		if text, e := bufio.NewReader(conn).ReadString('\n'); e ==nil {
			fmt.Println(text)
		}
	} else {
		fmt.Println("error", e)
	}
}
```
和使用net包的整体流程差不多，使用tls包的实现替换net包。
这样我们就实现了一个使用tls密钥加密的传输通道了。

## UDP ##
UDP的使用和TCP类似也包括一种加密的UDP传输。具体UDP和TCP的区别就不详细说了，这部分是考试面试的重点，一般都能说个123出来。

与TCP实现不一样的是，UDP要先使用`net.ResolveUDPAddr(string, string)`来获得一个`*net.UDPAddr`，然后使用`net.ListenUDP("udp", addr)`监听这个端口的UDP通信。一个简单实现如下：

```golang
import (
	. "fmt"
	"net"
	"os"
)

var MSG = ([]byte)("hello golong udp\n")

func main() {
	if addr, e := net.ResolveUDPAddr("udp", ":1026"); e == nil {
		if server, e := net.ListenUDP("udp", addr); e == nil {
			// infinit loop
			for buffer := MakeBuffer(); ; buffer = MakeBuffer() {
				if n, client, e := server.ReadFromUDP(buffer); e == nil {
					go func( c *net.UDPAddr, packet []byte) {
						if n, e := server.WriteToUDP(MSG, c); e == nil {
							Printf("%v bytes written to %v\n", n, c)
						}
					}(client, buffer[:n])
				} else {
					Println("error1", e)
				}
			}
		} else {
			Println("error2", e)
		}
	} else {
		Println("error3", e)
	}
}

func MakeBuffer() ([]byte) {
	return make([]byte, 1024)
}
```
作为server需要一个无限循环保持监听，每当收到一个客户端请求(`ReadFromUDP`)，就启动一个Goroutine服务这个请求(`WriteToUDP`)。

UDP不想TCP服务可以用telnet连接，因为它是一个无连接的通信，我们需要自己写一个请求的客户端。client也比较简单，其实现如下:

```golang
import (
	. "fmt"
	"net"
	"bufio"
)

var CRLF = ([]byte)("\n")

func main() {
	if addr, e := net.ResolveUDPAddr("udp", "localhost:1026"); e == nil {
		if server, e := net.DialUDP("udp", nil, addr); e == nil {
			defer server.Close()
			// 随意发送一些内容到server等待回应
			// 这里可以多次(for 循环写都可以)写内容到服务器
			if _, e := server.Write(CRLF); e == nil {
				if text, e := bufio.NewReader(server).ReadString('\n'); e == nil {
					Printf("%v", text)
				}
			}
		} else {
			Println("error2", e)
		}
	} else {
		Println("error3", e)
	}
}
```
这里注意注释里面的话，使用`Write()`方法向UDP Server发送一些内容，然后才会受到Server的返回，返回使用bufio读取。一个UDP的Dial实例可以多起发送数据和读取返回。

### 使用加密的UDP ###
Golang提供了很方便的加密UDP连接，只需要提供一个密钥，就可以对传输的数据进行加密了，当然我们需要对server和client的数据都进行修改。
rsa encrypt server:

```golang
import (
	. "fmt"
	"bytes"
	"crypto/rand"
	"crypto/rsa"
	"crypto/sha1"
	"encoding/gob"
	. "net"
)

var MSG = []byte("hello go scure udp")
var RSA_LABEL = []byte("server")

func main() {
	Serve(":1025", func(conn *UDPConn, c *UDPAddr, packet *bytes.Buffer) (n int) {
		var key rsa.PublicKey
		if e := gob.NewDecoder(packet).Decode(&key); e==nil {
			if resp, e := rsa.EncryptOAEP(sha1.New(), rand.Reader, &key, MSG, RSA_LABEL); e==nil {
				n, _ = conn.WriteToUDP(resp, c)
			}
		}
		return
	})
}

func Serve(addr string, f func(*UDPConn, *UDPAddr, *bytes.Buffer) int) {
	Launch(addr, func(conn *UDPConn) {
		for {
			buffer := make([]byte, 1024)
			if n, client, e := conn.ReadFromUDP(buffer); e == nil {
				go func(c *UDPAddr, b []byte) {
					if n := f(conn, c, bytes.NewBuffer(b)); n!=0 {
						Println(n, "write to ", c)
					}
				}(client, buffer[:n])
			}
		}
	})
}

func Launch(addr string, f func(*UDPConn)) {
	if a, e := ResolveUDPAddr("udp", addr); e == nil {
		if server, e := ListenUDP("udp", a); e == nil {
			f(server)
		}
	}
}
```

rsa used  client

```golang
import (
	. "fmt"
	"bytes"
	"crypto/rand"
	"crypto/rsa"
	"crypto/sha1"
	"crypto/x509"
	"encoding/gob"
	"encoding/pem"
	"io/ioutil"
	. "net"
)

var RSA_LABEL = []byte("server")

func main() {
	Connect("localhost:1025", func(server *UDPConn, priKey *rsa.PrivateKey) {
		cipherText := MakeBuffer()
		if n, e := server.Read(cipherText); e == nil {
			Println("cipher text", string(cipherText))
			// cipherText是加密的数据，需要解密
			if plainText, e := rsa.DecryptOAEP(sha1.New(), rand.Reader, priKey, cipherText[:n], RSA_LABEL); e == nil {
				Println("receive decrypt string:", string(plainText))
			}
		}
	})
}

func Connect(address string, f func(*UDPConn, *rsa.PrivateKey)) {
	LoadPrivateKey("key.pem", func(pk *rsa.PrivateKey) { // pk is private key
		if addr, e := ResolveUDPAddr("udp", address); e ==nil {
			if server, e := DialUDP("udp", nil, addr); e == nil {
				defer server.Close()
				SendKey(server, pk.PublicKey, func() {
					f(server, pk)
				})
			} else {
				Println("err1", e)
			}
		} else {
			Println("err2", e)
		}
	})
}

func LoadPrivateKey(file string, f func(*rsa.PrivateKey)) {
	if file, e := ioutil.ReadFile(file); e == nil {
		if block, _ := pem.Decode(file); block != nil {
			if block.Type == "RSA PRIVATE KEY" {
				if key, _ := x509.ParsePKCS1PrivateKey(block.Bytes); key !=nil {
					f(key)
				}
			}
		}
	} else {
		Println(e)
	}
	return
}

func SendKey(server *UDPConn, publicKey rsa.PublicKey, f func()) {
	var encodedKey bytes.Buffer
	if e := gob.NewEncoder(&encodedKey).Encode(publicKey); e == nil {
		if _, e = server.Write(encodedKey.Bytes()); e == nil {
			f()
		}
	}
}

func MakeBuffer() (r []byte) {
	return make([]byte, 1024)
}
```

需要注意的几点:

- RSA_LABEL 是rsa加密使用的一个label，需要客户端和服务器端一致
- 请求流程是客户端读取一个密钥，然后获取一个publickey，使用gob序列化后发送到server， server读取出key，使用这个publicKey对返回的数据进行加密
- 客户端读取返回后，使用密钥进行解密，读取明文
- 客户端代码使用了大量的回调，方法字面量，理解起来有点麻烦，可以用javascript的回调的思路去理解

done
