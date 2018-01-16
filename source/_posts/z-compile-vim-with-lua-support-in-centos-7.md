---
date: 2015/07/20 12:34:50
title: CentOS 编译安装vim 7.4添加Lua支持
categories:
- server
tags:
- vim
- tool
- linux
---

一些vim插件需要提供Lua支持，特别是常见的补全插件，前段时间安装的[yadr](https

## 下载源码 ##
首先，编译安装嘛，先下载源码， 可是呢这两sourceforge挂掉了，vim是托管在sf上的，导致下载页面也不能访问了，甚至vim的官网 www.vim.org 也不能访问了，幸好vim在github上有一个备份  [https://github.com/vim/vim](https://github.com/vim/vim "https://github.com/vim/vim") 或者直接访问vim.org的ftp站： [ftp://ftp.vim.org/pub/vim/unix/](ftp://ftp.vim.org/pub/vim/unix/ "ftp://ftp.vim.org/pub/vim/unix/")

```
// 使用下面之一的方法下载源码
// git下载的话体积会大一些，好处是以后可以方便地更新
git clone git@github.com:vim/vim.git
wget -O ftp://ftp.vim.org/pub/vim/unix/vim-7.4.tar.bz2
wget ftp://ftp.vim.org/pub/vim/unix/vim-7.4.tar.bz2

tar xjvf vim-7.4.tar.bz2
cd vim74
```
## 编译 ##
vim的编译其实很简单，就configure->make->make install 这样的流程。但是要添加 Lua支持，就有一些麻烦了。

configure的配置大概是这样的：

```
./configure --prefix=/data/vim74 --with-features=huge --with-luajit --enable-luainterp=yes
```

首先如果不在 configure配置那手动打开 `--enable-fail-if-missing` 这个选项，你会发现，configure没有问题，make没有问题，make install也OK，但是运行生成的vim： ` vim --version`会发现Lua前面还是一个"-"（表示没有Lua支持）
因为其实configure那根本就没有找到lua的支持，只是默认跳过了 `--enable-luainterp=yes` 选项。。。
所以应该这样运行configure：

```
./configure --prefix=/data/vim74 --with-features=huge --with-luajit --enable-luainterp=yes --enable-fail-if-missing
```

如果你的机器没有安装lua 和luajit的话会在检查lua支持那里中断了。

### 安装Lua和LuaJit ###
Lua的安装倒是比较方便，官方repo里面就有，当然要想安装新版的也可以自己编译安装。

```
sudo yum install lua lua-devel -y
```
luajit不在centos的官方repo里面，我们需要编译安装;

```
wget http://luajit.org/download/LuaJIT-2.0.4.tar.gz
tar -xzvf LuaJIT-2.0.4.tar.gz
cd LuaJIT-2.0.4
// 使用默认安装路径
make
sudo make install
```

## 编译安装 ##
安装好lua支持后，再运行configure就不会报错了

```
./configure --prefix=/data/vim74 --with-features=huge --with-luajit --enable-luainterp=yes --enable-fail-if-missing
make
sudo make install
```

## 运行 ##
运行编译的vim：

```
/data/vim74/bin/vim
```
可能你会发现这样的错误:

```
/data/vim74/bin/vim: error while loading shared libraries: libluajit-5.1.so.2: cannot open shared object file: No such file or directory
```
显然是安装的luajit有问题。我们找一下luajit这个.so文件在哪里

```
sudo find / -name libluajit-5.1.so.2
```

发现下面有这个文件：

1. /home/wuxu/tars/ngx_openresty-1.7.10.2/build/luajit-root/data/openresty/luajit/lib/libluajit-5.1.so.2
2. /usr/local/lib/libluajit-5.1.so.2
3. /data/openresty/luajit/lib/libluajit-5.1.so.2

显然1，3是openresty编译的，第二个才是我们安装luajit支持，我们需要给这个.so生成一个软链接，让vim能找到它：

```
sudo ln -s /usr/local/lib/libluajit-5.1.so.2 /lib64/libluajit-5.1.so.2
```

这样再运行vim就不会有问题了～也有Lua支持了，不过Vim的自动补全还没用过呢，学会了再做记录吧。
