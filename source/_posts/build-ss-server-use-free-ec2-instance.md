---
date: 2015/10/12 12:34:50
title: 在免费的Amazon EC2实例上搭建shadowsocks服务器
categories:
- server
tags:
- tools
- Liux
- centos
- ss
---

shadowsocks是非常方便的“出墙”方式，也是现在最流行最被推荐的方式。虽然shadowsocks的作者被逼把github上面的代码删除掉了，但是它依旧是最好用的反长城工具。

为了出墙，之前买了jandb的包年服务，但是最近经常出问题，很不稳定，像ss这种服务购买了出了问题基本都没地方反映，因为只能低调运行，也是很烦恼。今天早上又发现不能上Google了，正好前段时间申请的免费EC2（新用户都可以申请一个最低配置的VPC一年的免费试用）

如果还没有申请，建议在申请的时候看有没有JP或者HK或者SG的机房，我选的us-west的机房，速度实在有点慢。

免费的EC2实例只能选Liux，我选的是Redhat7的系统，因为一直用的CentOS和Fedora，比较熟悉。ss-server有很多种安装方式，有libev的包，有用python,nodejs,golang等多种语言实现。考虑到免费的实例配置很低，推荐使用libev的包。
<!-- more -->
安装ss-server有两种方式，一种使用社区的自动安装脚本，一种是编译安装，下面分别介绍。

## 使用脚本安装 ##
网站 [https://teddysun.com/357.html](https://teddysun.com/357.html "https://teddysun.com/357.html") 提供了自动安装脚本
`https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-libev.sh`

安装与管理都非常方便，推荐使用这种方式安装。

可以用下面几条命令安装：

```bash
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-libev.sh
chmod +x shadowsocks-libev.sh
sudo ./shadowsocks-libev.sh 2>&1 | tee shadowsocks-libev.log
```
安装过程中会询问端口密码等配置项，也可以使用默认值。

这种方式会自动添加开机启动服务，还会在init.d中添加脚本，可以使用systemctl工具启动、关闭和查看服务运行状态。

配置文件在 `/etc/shadowsocks-libev/config.json`，应该默认的配置就是可用的不需要修改。

```
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
查看状态：/etc/init.d/shadowsocks status
```

## 编译安装 ##
EC2的实例应该已经安装了很多基础的包，先安装下面的包，如果已经装过了yum会自动跳过。

```
sudo yum install build-essential autoconf libtool openssl-devel gcc -y

// install git
sudo yum install git -y

// download source code
git clone https://github.com/madeye/shadowsocks-libev.git
cd shadowsocks-libev

// configure, 默认安装在/usr/local/bin下，可以配置prefix
./configure && make
sudo make install
```

这样安装的好像没有默认的配置文件，可以在运行时指定参数：

```
nohup /usr/local/bin/ss-server -s your_ec2_public_ip -p 8981 -k admin888 -m aes-256-cfb &
```
其中 

- -s指定的ip是你在ec2的dashboard看到的public ip，不是ifconfig查到的ip，那是在Amazon cloud中的private ip.
- -k 指定密码
- -m 指定加密方法

关闭方式可以ps查看进行id然后kill掉。。。。就是这么粗暴。

## 注意防火墙策略 ##
需要额外注意的是EC2的防火墙策略。我一开始使用编译安装，但是客户端怎么都连接不到，查看系统的防火墙，iptables和firewalld都没有安装，运行后端口都在监听，但是在本地telnet到ss的端口都是不通的，后来才知道ec2的防火墙策略是在dashboard中配置的，叫做Security Groups。我们先添加一个Scurity Group把设定的端口添加进去，注意 source设置为任意，添加后到运行的实例的页面
![应用Security Group](/images/post/ec2-sg.png)
把新添加的策略勾选并assign。

这样应该就可以使用了。
