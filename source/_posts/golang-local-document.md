---
date: 2015/10/31 12:34:50
title: Go 启用本地文档
categories:
- programming
tags:
- network
- golang
- go
- document
---

Go很“先进”的一点在于它的文档和提供的文档工具 godoc ，但是由于golang的官网被 万能的GFW 干掉了，有时候要查文档很不方便。虽然翻墙也不麻烦，但是如果能有个本地的文档那就更方便了。

以往的本地文档都是以PDF或者CHM的形式存在，使用的应该都能感觉他们的缺点，PDF缺乏组织，查找/索引很不方便，而CHM限于Windows的技术支持，并且有时候也不是很方便。

实际上Go的安装包里提供了基础包的文档，他就保存在 `GOROOT/doc`（好像不是这个目录） 下，是一些html文件，当然我们可以直接浏览器打开，有一种更见好用的方式是用godoc提供的http服务，在命令行运行：

```
godoc -http:6060 &
// & 表示后台运行，windows下可能需要保持命令行窗口不关闭，如果要关闭窗口且保持服务运行
// 需要先运行 eixt 然后再关闭窗口，要关闭进程可以新开一个bash，ps找到PID后Kill
```
这条命令会监听6060端口的http请求，提供和golang.org同样的页面服务 。浏览器打开 `http://localhost:6060` 就可以方便的查看文档啦。


