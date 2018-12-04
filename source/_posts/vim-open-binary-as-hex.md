---
date: 2015/11/14 12:34:50
title: vim打开二进制文件
categories:
- Liux
tags:
- vim
- Liux
---

使用Vim编辑文本文件非常方便，但是打开一个二进制文件的时候会显示乱码，而不能像UltraEdit那样显示十六进制的数据，我们可以用一个小trick实现。
一般情况下，打开一个二进制文件如下图所示，我们打开一个 .o 文件

![](/images/Liux/vim-binary.png)

要让其显示为十六进制的，可以在命令模式键入下面的命令：

```
:%!xxd
```
这里是运行一个外部命令xxd

>  xxd - make a hexdump or do the reverse.

运行这个命令之后，vim显示如下：

![](/images/Liux/vim-hex.png)

如果要返回之前的现实，运行下面的命令

```
:%!xxd -r
```

这个trick的原理就是使用外部命令xxd生成一个hexdump。

完
