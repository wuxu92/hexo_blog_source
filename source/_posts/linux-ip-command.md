---
date: 2015/12/04 12:34:50
title: Linux ip/ss 命令详解
categories:
- server
tags:
- Liux
- centos
- nm
---
![](/images/Liux-logo.png)
CentOS 7正式废弃了很多旧的工具包，比如历史悠久的ifconfig, netstat等网络相关的命令，还有locate这些命令也没有了。新的系统使用ip, ss, net等命令替代了之前的网络操作命令。

新的命令设计的比较复杂，其帮助文档看起来都头晕，这里记录一下一下常见的操作。

## ip 命令
没有了ifconfig命令之后，ip命令是其替代品。ip命令的作用如下：

> ip - show / manipulate routing, devices, policy routing and tunnels

命令使用格式如下：

```
ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
       OBJECT := { link | addr | addrlabel | route | rule | neigh | ntable | tunnel | tuntap |
               maddr | mroute | mrule | monitor | xfrm | netns | l2tp | tcp_metrics }
       OPTIONS := { -V[ersion] | -s[tatistics] | -r[esolve] | -f[amily] { inet | inet6 | ipx |
               dnet | link } | -o[neline] }
```
可以看到二级命令非常多，很难记。最常用的是 `ip addr` 也可以简写为 `ip a`（ip命令中更有很多这种简写方法）。操作网络地址相关内容，比如列出ip地址，添加ip地址，删除ip等待。

```
ip addr show
# Only show TCP/IP IPv4
ip -4 a
# Only show TCP/IP IPv4
ip -6 a

# show eth0 interface
ip a show eth0

# show on running interface
ip link ls up

# add/del ip address
ip a add {ip_addr/mask} dev {intereface} [label label_name] // 可选的设置一个label
ip addr add 192.168.0.123/24 dev eth0

#remove ip address
ip a del {ip_addr} dev {interface}
ip addr del 192.168.0.123 dev eth0

# flush ip address; delete all the IP addresses matches
# 可以flush一个地址或者一个label标记的所有地址
ip a flush label "label"
ip -s a f to 192.168.2.0/24   // -s 输出统计信息 to limit to given IP address/prefix

# up or down a device
ip link set dev {interface} {up|down}
ip l set dev eth0 down
```

`ip link set` 命令可以设置很多的值，看一下自动补全提示：

```
$ ip l set <tab>
address     -- specify unicast link layer (MAC) ad
arp         -- change ARP flag on device
brd         -- specify broadcast link layer (MAC) 
broadcast   -- specify broadcast link layer (MAC) 
dev         -- specify device
down        -- change state do down
dynamic     -- change DYNAMIC flag on device
mtu         -- specify maximum transmit unit
multicast   -- change MULTICAST flag on device
name        -- change name of device
peer        -- specify peer link layer (MAC) addre
promisc     -- set promiscious mode
txqlen      -- specify length of transmit queue
txqueuelen  -- specify length of transmit queue
up          -- change state to up
```
一般用法都是 `ip link set {cmd} dev {interface}`。比如设置mtu：

```
ip link set mtu 3000 dev eth0
```

ip命令还能查看与邻近节点（neighbour）的可达性：

```
ip n show         // same as ip neigh show
ip n add {ip_addr} lladdr {MAC/LLADDRESS} dev {interface} nud {perm|noarp|stale|reachable}
ip n del {ip_addr} dev eth0
```
会输出附近节点的arp信息。可以手动添加这些arp条目。

路由表信息也由ip命令提供，使用route/r 子命令操作：

```
# list route table
ip r
ip r list 192.168.0.0/24
ip r add {default} {network/mask} dev {interface}
ip r add (default) {network/mask} via {gateway_ip}
ip r del default
ip r del network/mask dev wth0
```
ip命令远比上面介绍的部分复杂，这里不能一一列出了，使用一个带tab补全的终端非常有用，另外虽然ifconfig命令没有了，但是基于图形用户界面的 `nmtui` (network management text user interface) 还是预装了，可以用它来配置ip/DNS等。如果没有安装，使用下面的命令安装： `sudo yum install NetworkManager-tui -y`。

## ss 命令
ss是另一个很重要的工具，ss是socket statistics的缩写，用于代替之前使用netstat命令。ss能够显示比netstat更多的信息并且速度也更快。netstat是从 /proc 下的文件中读取信息再整理显示的，而 ss 命令直接从内核空间获取信息。

ss的man页面介绍如下：

> ss - another utility to investigate sockets

直接执行 ss 会列出当前所有已建立的非监听的（non-listening）连接，一个常用的参数 `-ntl`，参数意义为：

- `-n` --numeric，显示端口数字而不是服务名字，比如显示 80 而不是 http
- `-t` --tcp， 即显示 tcp 套接字，同理常用 -u 表示 udo 套接字
- `-l` --listening，也好理解，默认不显示监听的套接字，这个参数指明只显示监听中的套接字
- `-4` --ipv4也是常用的，在查看服务监听状态时，常指定 -4 或者 -6 结果更加清晰
- `-p` --processes,显示使用这个套接字的进程id，**这个参数需要 sudo 权限**
- `-s` --summary，显示套接字使用的统计信息
- `-o` --options，显示相关的时间信息

还可以更具套接字状态过滤输出，比如下面的命令：

```
ss -t4 state established
ss -t4 state time-wait
```
连接的状态有很多中，常用如下：

1. established
2. syn-sent
3. syn-recv
4. time-wait
5. closed
6. closing
7. all
8. connected

还可以通过指定dport和sport过滤输出：

```
# 还可以使用 or，666
ss -nt dst :443 or dst :80
// dport 大于1024的连接
ss -nt dst gt :1024  
```
当然更常用的过滤是 grep 啦，哈哈哈。

要监控网络流量的动态，可以用top相关命令，也可以用watch工具：

```
watch -n 1 "ss -t4"
```
这样每秒中会刷新一次ss的结果。

## 参考：

- [http://www.cyberciti.biz/faq/Liux-ip-command-examples-usage-syntax/](http://www.cyberciti.biz/faq/linux-ip-command-examples-usage-syntax/)
- [http://www.binarytides.com/Liux-ss-command/](http://www.binarytides.com/linux-ss-command/)
