---
date: 2015/07/28 12:34:50
title: nsq实时消息服务的TCP协议规范
categories:
- tools
tags:
- nsq
- golang
- mq
---

NSQ是使用golang开发的实时分布式消息处理平台，由bit.ly开源的项目，可以用来处理每天十亿级别的消息。

由于nsq可以直接编译为可执行的二进制文件的特性，nsq的安装非常简单，下载二进制包解压就可以运行了。 参考[官方文档](http://nsq.io/deployment/installing.html "http://nsq.io/deployment/installing.html")

快速运行也很简单依次在终端中启动nsq的服务即可。这里也不细说了，参考： http://nsq.io/overview/quick_start.html

关于nsq的几个组件会在后面记一篇，现在主要学习一些nsq中使用的TCP协议规范。

> NSQ的协议足够简单，任何语言编译客户端都很容易。

**nsqd进程通过监听配置的TCP端口来接受客户端连接**

**对nsq的用途理解有点错误了**
