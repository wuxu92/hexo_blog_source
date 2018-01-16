---
title: 服务器访问频率限制和IP限制
date: 2016-06-29 10:01:43
categories:
- server
tags:
- nginx
- linux
- web server
- lighttpd
---

最近公司部署在阿里云上的服务器被恶意攻击了，大概几百台肉鸡不停地向某个接口发送请求。由于一些各种原因，过多的请求导致服务器CPU占用率几乎是一直在100%。虽然应用勉强还能使用，但是这样被请求服务器真是压力非常大。

由于公司现在还没有专职的运维去处理这些请求所以只好自己动手先做一些限制，大概两个思路：

1. 对接口的请求限速
2. 2.封禁一批IP

## 基础环境
先记录一下服务器的基本环境，双核，4G内存，Ubuntu 14.04，现在的web server是lighttpd，数据库是mysql。

## 接口限速
由于现在部署的代码使用旧的框架，本身没有对接口限速的功能，只能从web server方面入手。

现在的web server是lighttpd，lighttpd的特点是轻快，内存占用小，但是模块化不是很好，似乎在lighttpd上面做 rate limiting 不是那么方便，我觉得可以使用使用一个围魏救赵的方案：

搭建一个nginx作为前端server，在nginx这里做rate limit，nginx把请求发送到lighttpd，lighttpd不对外暴露端口，只接受来自nginx的请求。

在lighttpd的配置文件(/etc/lighttpd/lighttpd.conf)中加上

```
server.port = 80001
```
重启lighttpd，记得在防火墙中Drop所有8001端口的请求。

安装好nginx（推荐编译安装openresty）后，配置转发如下：

```
upstream lighttpd {
	server 127.0.0.1:8001
}

server
{
	...
	location / {
		proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
        proxy_pass http://lighttpd;
	}
}
```
启动nginx后，原来所有的请求都会发送到nginx了，而对用户来说，感觉不到任何的变化。

有了nginx之后我们可以使用 [ngx_http_limit_req_module](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html) 这个模块做限速了，具体内容可以参考nginx的文档。


### 定义限制请求域
rate limit首先要定义一个或多个zone，然后在特定的context使用它们。这些定义一般定义在http上下文中。

```
http ｛
	## 对每一个IP的请求限制为1次每秒
	limit_req_zone $binary_remote_addr zone=perip:10m rate=1r/s;

	## 对一个server_name下的请求限制为10次每秒（这个数值有点低，一般会改大很多）
	limit_req_zone $server_name zone=perserver:10m rate=10r/s;
}
```

### 使用请求限制
可以在 http, server,location 上下文使用上面定义的域，使用方法如下：

```
server {
	...
	limit_req zone=perip burst=5 nodelay;
    limit_req zone=perserver burst=10;

	location /api {
		limit_req zone=perip burst=3 nodelay;
	}
}
```
限速的配置是可继承的，当前级别没有配置就会从父级配置中继承：

> These directives are inherited from the previous level if and only if there are no limit_req directives on the current level

关于 nodelay的配置，说明如下：

> If delaying of excessive requests while requests are being limited is not desired, the parameter nodelay should be used

## IP 限制
第二个思路是找出频繁请求的那些IP，在防火墙或者 web server 级别做限制，拒绝它们的请求。

首先通过请求日志（access_log）找出请求次数最多的前50个IP：

```
awk '{print $1}' /path/to/web-server-log/access.log | sort|uniq -c | sort -nr | head -n 50
```
我只是在nginx或者lighttpd层做限制，iptables在我手上很容易玩坏，不是很熟悉，就在web server做配置了：

### nginx限制IP访问
nginx的配置很简单，在指定上下文deny掉指定的IP就可以了，如下：

```
location / {
    deny  192.168.1.2;
    allow 192.168.1.1/24;
    allow 127.0.0.1;
    deny  all;
}
```

### lighttpd限制IP访问
lighttpd的限制也非常方便，只不过要先对上面获得的50个ip做一些简单的处理，然后在ilghttpd.conf中加入下面的配置：

```
$HTTP["remoteip"] != "1.231.2.1|23.24.54.213|ip-3|ip-4|more-ip" {
 url.access-deny = ( "" )
}
```

## 总结
第一种方法应该是更好的，或者说是必须的，但是我调了一下，限速的参数可能调太小了，导致正常的访问都不太好用了，之后需要继续调优参数。

第二种限制IP的方法也可以有，可以用一个计划任务隔一段时间就跑一批IP扔到禁止列表中，不过要是能加到iptables中，对服务器的压力就更小了，这个后面可以尝试一下。