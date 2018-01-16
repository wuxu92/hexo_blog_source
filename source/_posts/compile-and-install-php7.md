---
date: 2015/12/03 12:34:50
title: 编译安装PHP7正式版
categories:
- server
tags:
- php
- lang
- php7
---
![](/images/php.jpg)

PHP 7正式版终于在今天发布了，<del>虽然在官网 [php.net](php.net) 还没有挂出链接（原来是因为中国时区比较早，现在已经有下载了），但是在鸟哥的微信公众号中给出来下载地址</del>，在github上也有了 7.0.0  的release。不知道为什么github上面的状态一直显示 build error。但是下载源文件编译是成功的。基本的安装步骤和之前编译安装 beta1 是一样的。

官方的下载地址： [http://www.php.net/downloads.php](http://www.php.net/downloads.php)

可以参考之前的文章： [http://wuxu92.github.io/compile-and-install-php7-beta1/](http://wuxu92.github.io/compile-and-install-php7-beta1/) 唯一的区别是正式版没有了 `--with-mysql`  选项，因为正式版已经没有旧的mysql函数的支持了，需要使用 mysqli 或者 PDO 替代早已不建议使用的mysql函数。

下载链接：  [http://cn2.php.net/get/php-7.0.0.tar.gz/from/this/mirror](http://cn2.php.net/get/php-7.0.0.tar.gz/from/this/mirror)

<!-- more -->
## 下载源码 ##
```
wget http://cn2.php.net/get/php-7.0.0.tar.gz/from/this/mirror
// 这条命令从国内的镜像下载，下载的文件会命名为 mirror
// 需要 mv mirror php-7.0.0.tar.gz 重命名
or
wget http://php.net/distributions/php-7.0.0.tar.gz
```
解压源码： 

```
tar xzvf php-7.0.0.tar.gz
cd php-7.0.0/
```

## 安装必要的工具 ##
同样我们要先安装一些基础软件，如果之前没有安装过则需要安装。

```
sudo yum install -y gcc gcc-c++  make zlib zlib-devel pcre pcre-devel  libjpeg libjpeg-devel \
    libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel \
    glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel\
    e2fsprogs e2fsprogs-devel krb5 krb5-devel openssl openssl-devel \
    openldap openldap-devel nss_ldap openldap-clients openldap-servers \
    php-mysqlnd libmcrypt-devel  libtidy libtidy-devel recode recode-devel libxpm-devel
```

## 配置编译选项 ##
先buildconf，需要事先安装了autoconf，再在源码目录运行buidconf脚本。

```
sudo yum install -y autoconf
./buildconf
```
configure脚本参数，使用下面的配置，编译的php基本就满足使用了（和编译beta版本一样，只是去掉了 `--with-mysql` ）：

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
    --with-pdo-mysql=mysqlnd \
    --with-gettext=/usr \
    --with-zlib=/usr \
    --with-bz2=/usr \
    --with-recode=/usr \
     --with-xmlrpc \
    --with-mysqli=mysqlnd
```
和以往一样：

1. 最好不要使用默认安装路径，加上 ```--prefix=/path/to/install``` 参数
2. mysql相关的配置加上
3. curl, fpm相关的要enable

## 编译安装 ##
这步非常简单

```
make -j 4
make test
sudo make install
```
对于多核机器使用 -j 参数可以利用多个核，速度更快。如果安装路径是root权限的话，需要`sudo make install`， 因为安装需要创建新的文件

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

## 来自鸟哥的优化建议
php7发布之后，鸟哥在微信公众号更新了[一篇文章](http://mp.weixin.qq.com/s?__biz=MzIwNDExMjIzNA==&mid=401084310&idx=1&sn=82e16e3096862fc2ca460072bd7350d7&scene=23&srcid=1204VvYIHoePMaPt2VWSNdsh#rd)，提到了一些提高php性能的建议，这里记录一下。

### 开启opcache
记得启用Zend Opcache, 因为PHP7即使不启用Opcache速度也比PHP-5.6启用了Opcache快, 所以之前测试时期就发生了有人一直没有启用Opcache的事情. 启用Opcache非常简单, 在php.ini配置文件(如果编译是指定了config-file-path则php.ini在那个目录下)中加入:

```
zend_extension=opcache.so
opcache.enable=1
opcache.enable_cli=1
```
### 使用新的GCC
使用新一点的编译器, 推荐GCC 4.8以上, 因为只有GCC 4.8以上PHP才会开启Global Register for opline and execute_data支持, 这个会带来5%左右的性能提升(Wordpres的QPS角度衡量)

### 开启
在服务端开启HugePages, 然后开启Opcache的huge_code_pages.以我的CentOS 6.5为例, 通过 `sudo sysctl vm.nr_hugepages=512` 分配512个预留的大页内存，然后在php.ini中:

```
opcache.huge_code_pages=1
```
这样一来, PHP会把自身的text段, 以及内存分配中的huge都采用大内存页来保存, 减少TLB miss, 从而提高性能.

###开启Opcache File Cache(实验性), 
通过开启这个, 我们可以让Opcache把opcode缓存缓存到外部文件中, 对于一些脚本, 会有很明显的性能提升.在php.ini中配置:

```
opcache.file_cache=/tmp"
```

几张图：

![](/images/post/test-result.png)
make test 的结果，有一些测试没有通过

![](/images/post/make-install.png)
make install的结果

![](/images/post/info.php.png)
最后的phpinfo的输出

今天只是编译安装，关于PHP7的新特性，之后有机会再写吧。

## 完 ##