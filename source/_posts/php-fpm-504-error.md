---
date: 2015/09/18 12:34:50
title: PHP-FPM出现504错误
categories:
- server
tags:
- php
- linux
- http
---

504是一种常见的服务器错误，今天公司的开发机的服务器就出现了504错误，今天就要离职了，一天都没做什么事，正好这件事博哥找我看一下，查一下nginx的日志，发现是upstream timeout，我们知道nginx+php的模式下，nginx作为server把php的请求发到php-fpm进行处理，再把php-fpm处理的结果返回。
这里可以把php-fpm看作nginx日志里说的那个upstream。那么可以推出是php-fpm服务出了问题。

查看php-fpm服务的状态:

```
sudo service php-fpm status  # centos 6
sudo systemctl status php-fpm # centos 7
```
发现php-fpm的状态是running，没有问题～那是怎回事呢，不管三七二十一先 `sudo service php-fpm restart` （centos 7使用systemctl）试一下，发现竟然好了。

看起来问题是解决了，页面响应速度恢复了，但是一登录玉米平台，点开两三个页面有开始504了。。。
看来应该是服务器资源的问题，但是开发机配置很好，top一下可以看到资源是够用的，那就是php-fpm配置的问题了。

```
sudo vim /path/to/php-fpm.conf
```
找到 `pm.max_children` 的部分。可以看一下注释部分：

> ; The number of child processes to be created when pm is set to 'static' and the
> ; maximum number of child processes when pm is set to 'dynamic' or 'ondemand'.
> ; This value sets the limit on the number of simultaneous requests that will be
> ; served. Equivalent to the ApacheMaxClients directive with mpm_prefork.
> ; Equivalent to the PHP_FCGI_CHILDREN environment variable in the original PHP
> ; CGI. The below defaults are based on a server without much resources. Don't
> ; forget to tweak pm.* to fit your needs.
> ; Note: Used when pm is set to 'static', 'dynamic' or 'ondemand'
> ; Note: This value is mandatory.
> pm.max_children = 5

默认的配置值是5，公司开发机配置了一大堆的虚拟机，估计得20来个了，可以预测是child processes不够导致的504，可以根据服务器配置适当地调大这个参数，我把它设置为32了，应该可以适应当前的需求了。

关于504 bad gateway可以理解一下，php-fpm的全称是php fastcgi proceesses manager, cgi则是common gateway interface即通用网关接口，gateway timeout就可以理解为这个cgi超时没有及时返回给web server数据。

-----
刚刚发现这样改了还是会有504 timeout......囧了个囧，勇哥和我机智地分析了一下应该是代码中的问题，他们使用了curl请求另外的资源，我们猜测是curl长时间没有返回而导致php-fpm超时，修改代码设置curl请求的超时时间，再重启php-fpm请求，问题解决了。

一个问题出现可能有很多原因，要一点一点去分析，慢慢才能解决问题～～
