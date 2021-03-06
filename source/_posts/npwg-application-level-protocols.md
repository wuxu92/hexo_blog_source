---
date: 2015/11/07 12:34:50
title: Go 应用层协议
categories:
- programming
tags:
- go
- golang
---
![](/images/golang/gopher-banner-small.jpg)

> 这两天状态不太好，这一篇写的很糟糕
 
在端到端的通信中，是两个程序之间的通信，除了TCP/UDP这样的传输层协议外，我们需要应用之间的通信的协议，这样才能利用/理解传输的数据，这就是我们这一篇的应用层协议。
## 设计 ##
应用之间的通信协议设计要考虑下面的问题：

- 广播还是点对点？广播必须是UDP，本地多播或者更加实验性质的MBONE，点对点则可以是UDP和TCP
- 有状态还是无状态的？有状态的连接对应用更简便，但是要考虑连接奔溃的时候怎么办
- 传输层协议是否是可靠的？可靠连接一般更慢，但是就不需要担心丢包了，在内网的话UDP的可靠性也是非常高的
- 是否需要回复(reply)?接收端是否需要给发送方返回一个收到确认，如果要的话要考虑确认丢失，可能要使用超时
- 使用什么样的数据格式？通常会使用MIME规定的类型，或者使用字节编码数据
- 是否有突发性大流量的情况？这要考虑QoS(服务质量)，如果突然流量爆发如何处理。
- 多个流是否有同步的要求？比如视频和音频需要同步时间轴
- 是一个独立的应用还是为其他人提供的库？

设计一个应用层协议，上面是最基本要考虑的点。

## 数据格式 ##
消息的数据有两种主要的数据格式，一种是字节编码(byte encoded)，一种是字符编码(character encoded).

### 字节格式 ###
使用字节编码格式的数据一般使用第一个字节来区分消息类型。消息的处理器（handler）通过检查第一个字节转向不同的处理流程(不同的消息对应不同的请求)。其他字节是根据某种规定的规则序列化的数据体。

使用字节的优势是数据密度高也因此速度很快，其缺点是数据不透明，这其实也算一个好处，这样别人就难以逆向工程了。如果出现错误将很难被发现，很难调试。常用使用这种格式的是DNS、NFS到Skype。

针对字节数据，Conn 类型提供了读取字节的方法：

```
func (c Conn) Read(b []byte) (n int, err os.Error)
func (c Conn) Write(b []byte) (n int err os.Error)
```
### 字符格式 ###
使用base64编码后或者将二进制流转变成7-bit使用ascii字符发送。
字符格式通常使用多行形式的消息。第一行通常代表消息类型，剩余的数据是消息体数据。常使用行处理的函数。
在Go中没有管理字符流的直接支持，只能手动执行“换行”，要注意的是不同操作系统的换行很不一样， Unix系统使用'\n' 代表换行，Windows系统使用"\r\n"换行，**在internet上"\r\n"更加常用**。

## 状态管理 ##
应用需要管理状态信息来简化当前的操作，可以使用状态来保存应用信息。状态可以由服务器管理也可以由客户端管理。
可以使用应用状态转移图表设计状态系统。状态转移图可以跟踪应用的状态以及何时转移到新的状态。

> 这一章主要是概念性的东西，状态不太好，记的很乱，ごめ
