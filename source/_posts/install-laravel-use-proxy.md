---
date: 2015/09/03 12:34:50
title: laravel安装
categories:
- framework
tags:
- php
- framwork
- laravel
---

之前一直使用Yii作为主力的PHP框架，但是也有很多人推荐了解学习一下laravel，虽然很多laravel的粉丝宣传的挺过的，但是也准备了解一下laravel。

官方提供的laravel的安装方式是通过composer进行安装，这是php发展的趋势，也是一个好的方向。但是在国内，这种通过网络安装的方式经常会有一些问题。特别是在校内的一台没有联网的机器就更麻烦了，幸好要安装的机器可以通过代理连接网络，这样问题也许还可以解决。

要安装laravel的最新版本，分以下几步： 1.安装composer， 2. 安装laravel

## 安装composer ##
在composer的[官网](https://getcomposer.org/)有详细的资料介绍如何安装，一般使用下面的方式：

```
curl -sS https://getcomposer.org/installer | php -- --filename=composer
```
这里要为curl设置代理：

```
curl -sS -x prox_host:port https://getcomposer.org/installer | php -- --filename=composer
```
这样好像就可以了。其实直接下载一个[composer.phar文档](https://getcomposer.org/composer.phar "https://getcomposer.org/composer.phar")也可以，还更加方便呢，在windows 下建议直接下载phar文档，使用官方的exe安装器反而因为网络原因会安装失败。

如果使用命令安装的话可以直接在终端运行composer了，如果下载的是phar归档，则需要添加可执行权限，然后直接运行该归档

```
chmod +x composer.phar
./composer.phar
```

## 安装laravel ##
laravel的安装方式有两种：

1. 通过composer安装laravel-installer，再通过installer新建项目
2. 通过composer create-project 直接新建项目。

### laravel installer ###
我最开始使用第一种方法，先安装laravel-installer：

```
composer require "laravel/installer=~1.1"
```
要配置代理需要配置环境变量，也可以指直接在bash设置：

```
set http_proxy=host:port
set https_proxy=host:port
```
再运行，可能会报`[ErrorException] zlib_decode(): data error` 这样的错误，这是openssl验证的问题，可以的通过：

```
php -r "print_r(openssl_get_cert_locations());" 
```
查看openssl的配置情况并修改配置，但是这样很麻烦，我们可以直接关闭ssl的验证，这环境变量中设置:

```
HTTP_PROXY_REQUEST_FULLURI=false
HTTPS_PROXY_REQUEST_FULLURI=false
```
这是再安装就不会有错了。
上面的方法会把laravel installer安装到当前目录下的vendor/laravel。可以把这个目录添加到PATH，也可以直接全目录运行。

```
laravel new blog
```
这个命令会通过 Symfony的组件下载文件并新建一个laravel的目录，我在这一步不知道怎么设置代理，第一次网络连接失败，后来一次被我Ctrl-c了。

### composer create-project ###
由于前面把composer配置得可用了，就想应该可以直接使用composer 新建项目了。就使用

```
composer create-project laravel/laravel --prefer-dist
````
这样就可以新建一个项目了。文件夹名字为laravel，应该可以自己修改了，剩下的配置一个vhost就可以运行laravel了。
