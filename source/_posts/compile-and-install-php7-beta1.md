---
date: 2015/07/19 12:34:50
title: 编译安装PHP7 beta1
categories:
- server
tags:
- php
- lang
- php7
---

前段时间去php.net上闲逛发现PHP7的beta1版本出来了，就尝试着编译安装了一下

之前在鸟哥的博客里看到了他们做的PHP的性能测试，相对于PHP5.6都是有很大的提升的，并且PHP的主版本号已经是2004年发布PHP5后，11年来首次更新，肯定PHP７是有很大的改变的，下面那就来试试PHP7的beta版吧，正式版大概在今年年底会发布。

## 下载源码 ##
测试版本的PHP下载页面： [https://downloads.php.net/~ab/](https://downloads.php.net/~ab/ "https://downloads.php.net/~ab/")

php7 beta1: [https://downloads.php.net/~ab/php-7.0.0beta1.tar.gz](https://downloads.php.net/~ab/php-7.0.0beta1.tar.gz "https://downloads.php.net/~ab/php-7.0.0beta1.tar.gz")

```
wget https://downloads.php.net/~ab/php-7.0.0beta1.tar.gz
或者
curl -O https://downloads.php.net/~ab/php-7.0.0beta1.tar.gz
```
<!-- more -->
解压源码： 

```
tar xzvf php-7.0.0beta1.tar.gz
cd php-7.0.0beta1/
```

## 安装必要的工具 ##

```
sudo yum install -y gcc gcc-c++  make zlib zlib-devel pcre pcre-devel  libjpeg libjpeg-devel \
	libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel \
	glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel\
	e2fsprogs e2fsprogs-devel krb5 krb5-devel openssl openssl-devel \
	openldap openldap-devel nss_ldap openldap-clients openldap-servers \
	php-mysqlnd libmcrypt-devel  libtidy libtidy-devel recode recode-devel libxpm-devel
```
这里面可能有一些已经安装过了，或者其实不是不需要的，因为一些功能我们可能会在编译时排除掉，但是像libxml2, zlib, freetype, bzip2, curl,curl-devel, openssl这些常用的包还是装上比较好，

## 配置编译选项 ##
configure脚本参数，使用下面的配置，编译的php基本就满足使用了：

```
./configure \
    --prefix=/data/php7 \
    --with-config-file-path=/data/php7/etc \
    --enable-mbstring \
    --enable-zip \
    --enable-bcmath \
    --enable-pcntl \
    --enable-ftp \
    --enable-exif \
    --enable-calendar \
    --enable-sysvmsg \
    --enable-sysvsem \
    --enable-sysvshm \
    --enable-opcache \
    --enable-fpm  \
    --enable-session \
    --enable-sockets \
    --enable-mbregex \
    --with-fpm-user=vagrant  \
    --with-fpm-group=nogroup \
    --enable-wddx \
    --with-curl \
    --with-mcrypt \
    --with-iconv \
    --with-gd \
    --with-jpeg-dir=/usr \
    --with-png-dir=/usr \
    --with-zlib-dir=/usr \
    --with-freetype-dir=/usr \
    --enable-gd-native-ttf \
    --enable-gd-jis-conv \
    --with-openssl \
    --with-mysql=mysqlnd \
    --with-pdo-mysql=mysqlnd \
    --with-gettext=/usr \
    --with-zlib=/usr \
    --with-bz2=/usr \
    --with-recode=/usr \
     --with-xmlrpc \
    --with-mysqli=mysqlnd
```

另一套configure(公司环境)

```
./configure --prefix=/data/php --with-config-file-path=/data/php/etc --with-mysql=mysqlnd --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-gd --with-iconv --with-zlib --enable-xml --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-mbregex --enable-fpm --enable-mbstring --enable-ftp --enable-gd-native-ttf --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --enable-session --with-mcrypt --with-curl --with-jpeg-dir --with-freetype-dir --with-xpm-dir=/usr --with-bz2
```
可能在运行配置脚本的时候会报一些错误，一般是缺少一些包，分析一下，把缺少的包安装上就可以了。注意几点：

1. 最好不要使用默认安装路径，加上 `--prefix=/path/to/install` 参数，毕竟这是beta版本的
2. mysql相关的配置加上
3. curl, fpm相关的要enable

## 编译安装 ##
这步非常简单

```
make
sudo make install
```
如果安装路径是root权限的话，需要`sudo make install`， 因为安装需要创建新的文件

## 配置文件 ##
机器不是很旧的话，编译和安装都是比较快的，安装完后需要设置一下配置文件，假设我们的安装路径是`/data/php7/`，在php7源码的解压目录运行下面的命令:

```
sudo cp php.ini-production /data/php7/etc/php.ini
sudo cp sapi/fpm/init.d.php-fpm /etc/init.d/php7-fpm
sudo cp /data/php7/etc/php-fpm.conf.default /data/php7/etc/php-fpm.conf
sudo cp /data/php7/etc/php-fpm.d/www.conf.default /data/php7/etc/php-fpm.d/www.conf
```
命令都比较易懂，就不细说了，主要是第2条复制启动脚本挺有用的。可以自己试试用systemctl控制起停。
复制过去的启动脚本可能没有执行权限，还需要手动添加权限:

```
sudo chmod +x  /etc/init.d/php7-fpm
```

## 完 ##
