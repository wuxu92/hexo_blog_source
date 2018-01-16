---
title: 使用tsocks让程序走socks代理
date: 2017-02-08 22:07:43
categories:
- liux
tags:
- linux
- fedora
---
很多时候，我们需要让联网程序走代理才能正常工作，但是命令行工具的代理设置通常比较麻烦，不像浏览器一样有一个设置界面，通常通过设置环境变量`HTTP_PROXY`来实现，有的工具通过配置文件或者命令行参数也可以实现。

但是这常见的多数是针对http代理的，而我们的特殊网络环境下使用socks5代理很常见的，而要走socks代理的话就更麻烦了。像有时候要从github clone项目的时候，git的socks代理就非常麻烦，在windows下可以用全局代理工具去实现，在linux下也需要一个方便的全局代理的工具。

tsocks（transparent socks）就可以满足我们的需求，tsocks可以方便地配置针对目的ip的代理服务器，可以将http请求代理到socks服务器，有方便的开关控制。在fedora下执行`sudo dnf install tsocks`就可以了，但是默认安装的没有将配置文件拷贝到`/etc/`目录下。需要手动拷贝一下：

```
sudo cp /usr/share/doc/tsocks/tsocks.conf.complex.example /etc/tsocks.conf
```
这样就可以了，可以在里面配置path和server的信息，配置项目都有说明吗，比较好懂。

使用的话，直接运行`tsocks`会启动一个走代理的shell，在这个shell里面运行的命令会根据配置文件的配置走socks代理了，`Ctrl+d`或者`exit`退出该shell就回到正常shell了。也可以直接在 tsocks 后面直接加要运行的命令， 如 `tsocks git clone git@github.com:***/**` 这样git就能走socks代理了。

常见的命令`tsocks show`查看当前shell是否有tsocks代理，`tsocks on|off`开关tsocks。

配合shadowsocks就可以实现几乎任意程序的自由上网了。