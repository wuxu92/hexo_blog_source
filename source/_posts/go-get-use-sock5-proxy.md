---
title: 让go get命令使用socks5代理
date: 2016-01-13 20:33:54
categories:
- go
tags:
- go
- gfw
---
不知道是因为什么鬼打墙的原因，GFW把go的官网给禁了，也许因为Go出身Google吧，真是*了狗了。不仅是 [golang.org](http://golang.org) 被墙了，现在Go的包管理站 [http://gopkg.in](http://gopkg.in) 也被墙了。直接用 `go get gopkg.in/package` 半天都没反应，只好用代理了。

如果是用http代理能访问的话，可以设置系统变量 `http_proxy` 就可以让 `go get` 走代理了，如果你有可以翻墙的http代理，可以用下面的方法：

```bash
http_proxy=http://ip:port go get gopkg.ig/package.v1
```

本地使用 shadowsocks做代理，因为ss是socks5代理，所以不能用上面的方法，go get又没有直接设置代理的地方，所以只好祭出代理利器：[Proxifier](https://www.proxifier.com/) ，这个工具有30天试用期，在国内也很容易找到破解版本。

打开Proxifier后添加 proxification rule，将 Applications设置为 `C:\Go\bin\go.exe`，这样重新打开终端（我的git bash），直接

```
go get gopkg.in/package.v1
```
就可以把包down下来了，虽然速度可能有点慢。

go相关的很多资源都被墙了，FK