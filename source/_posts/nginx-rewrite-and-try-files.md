---
date: 2015/08/27 12:34:50
title: nginx重定向的几点[人生的]经验
categories:
- server
tags:
- nginx
- linux
---

现在越来越多的server使用nginx做前端，在使用php的项目中越来越多的使用单一入口文件，而且很多时候希望隐藏这个入口文件，生成一个漂亮而简洁的url。以前在apache下是使用一个独立的rewrite模块，或者使用.htaccess文件实现重定向，nginx中需要小小的配置一下。

比如对于一个请求： http://localhost/index.php?r=c/a&p=v 这样，我们希望请求到 http://localhost/c/a?p=v。

比较老的做法是配置一个rewrite，以前的项目一般都是这么做的：

```
location / {
	if ($request_uri !~ "/(index\.php)") {
        rewrite ^/(.*)$ /index.php/$1 last;
    }
}
```
但是这样做有可能会导致循环重定向而返回500错误页面。并且有一个问题，就是静态文件也会被转发到index.php。
静态文件转发到php-fpm不仅导致做了无用的工作，而且对于允许用户上传图片/文件的应用可能导致安全问题。这是非常不推荐的用法。

为了解决上面的问题，我试过一个稍微麻烦但是能正常工作的方案：

```
if ($request_uri !~ "/static(.*)" ) {
  set $test  A;
} 
if ( !-e $request_filename) {
  set $test "${test}B";
} 
if ($test = AB) {
  rewrite ^/(.*) /index.php/$1 last;
}
```
这里假设静态文件都存放在/static目录下。这样可以让静态资源文件不被转发。但是这个方案啊，总觉得不那么优雅。

今天在解决一个问题的时候正好又查了一下，发现现在更多的使用的try_files配置来做重写。try_files的语法如下：

```
try_files file ... uri
try_files file ... = code
```
其作用域是server 和location，并且不能放在if条件里面。最常用的用法是：

```
try_files $uri $uri/ index.php
```
下面简单解释一下，try_files顾名思义就是尝试读取文件，正是对于请求的脚本不存在的情况，给nginx一个尝试读取脚本的策略。第一个是`$uri`就是读取uri指定的文件，如果不存在就把请求的看作目录，查找目录下有没有默认index文件（一般配置为index.html, index.htm, index.php）;如果有则读取这个文件。对于try_files的最后一个参数，会作一个 内部重定向/fallback，这个内部重定向可以看作一个内部子请求，会重新被nginx配置match一遍。注意，**只有最后一个参数会发起子请求**

在我们的配置里面最后一个参数是index.php这样，会发起一轮新的match会被nginx配置里面的 `location ~ .*\.(php|php5).*$ {}` catch然后进行转发到php-fpm解析。
当请求是静态文件时，因为能直接match的$uri，所以直接就返回静态文件的内容了，对于页面的请求就会转发到index.php.
这样就解决了index转发和静态文件不转发的问题，键值优雅得多了。但是有一个问题,那就是参数。

对一般的web框架来说，其请求的url一都是这样样式的：

```
http://host.com/index.php?r=controller/action&param1=value1&...
// 要使用下面的url访问
http://host.com/controller/action?param1=value1&...
```
需要注意的是，nginx在匹配try_files的最后一项时不会自动把args(也就是querystring)转发出去，需要手动加上，所以对于上面的需求，可以使用下面的配置：

```
try_files $uri $uri/ /index.php?r=$uri&$args
```
这样，对于`http://host.com/user/login?username=wuxu&passwd=123`，$uri匹配到`user/login`，$args匹配到`username=wuxu&password=123`,经过try_files配置之后就是`http://host.com/index.php?r=user/login&username=wuxu&password=123`了。（当然这里只是用login做示例，实际中的login可不能用GET传递参数哦）。

对于我们自己实现的框架可能有不同的路由方法，相应的修改try_files的策略就可以了。


当然try_files的最后一个参数还可以更复杂，具体可以看nginx的参考文档。

完。

参考: 

- [http://wiki.nginx.org/NginxHttpCoreModule#try_files](http://wiki.nginx.org/NginxHttpCoreModule#try_files "http://wiki.nginx.org/NginxHttpCoreModule#try_files")
- [https://servers.ustclug.org/2014/09/nginx-try_files-fallacy/](https://servers.ustclug.org/2014/09/nginx-try_files-fallacy/ "https://servers.ustclug.org/2014/09/nginx-try_files-fallacy/")
