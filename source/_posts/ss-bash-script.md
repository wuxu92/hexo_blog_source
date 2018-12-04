---
title: Shadowsocks的客户端脚本
date: 2016-10-10 21:55:47
categories:
- Liux
tags:
- Liux
- zsh
---
之前一直使用shadowsocks-qt5的ss客户端，但是最近不知道怎么回事，一启动它过几秒就崩溃,重装也没有效果，只好写两个脚本控制了。

首先需要安装sslocal的包：

```
sudo dnf install python-pip
sudo pip install shadowsocks
```
安装了shadowsocks后就可以用 `sslocal`命令来连接shadowsocks服务器了，还需要一个关闭连接，我们可以使用`ps -aux|grep` + `kill -9 pid` 来实现。具体的脚本见下：

## 启动脚本
启动要考虑关闭之前存在的连接，并且我的sever同时有ipv4和ipv6服务，可通过命令行参数选择连接到ipv6。具体脚本如下：

```
#! /bin/sh

ip=*7*.***.*7.*7
ip6=2**:f7**:4*:c**::****:****

if [[ $# -eq 1 && $1 -eq 6 ]]; then
    ip=$ip6
fi

# close existed sslocal proc
# may need change the path, or add bash to PATH
~/bin/bashes/close-ss
echo "connecting at $ip"
sslocal -s $ip -p 18989 -l 1080 -k password -t 600 -m aes-256-cfb &>>/path/to/log/ss.log &

echo "done with: $?"
```

在连接脚本中使用了关闭功能，因为对于一个新的连接需要先关闭之前的连接

## 关闭连接
关闭连接使用ps查找到pid，再调用kill关闭进程，也可以将pid写到一个文件，都差不多，具体如下：

```
#! /bin/sh
procs=`ps aux|grep sslocal`
count=`echo "$procs"|wc -l`

if [[ count -eq 1 ]]; then
    echo "no ss-local runs"
    exit;
fi

pid=`echo "$procs"|head -1 |awk '{print $2}'`
echo "closing by pid: $pid"

kill -9 $pid

if [ $? -eq 0 ]; then
    echo "result: success"
	# clean log file
    : > /path/to/log/ss.log
else
    echo "fialed to close sslocal"
fi
```

++ update @2017-02-10 ++
关闭sslocal的时候，之前使用的是ps+grep找出pid，实际上可以用pgrep更快更方便地找出来，更新后的代码如下：

```
#! /bin/sh
procs=`pgrep sslocal`
if [[ -z $procs ]]; then
    echo "no ss-local runs"
    exit;
fi

for pid in $procs; do
    echo "closing by pid: $pid"
    kill -9 $pid
done

if [ $? -eq 0 ]; then
    echo "result: success"
    : > ~/tmp/ss.log
else
    echo "fialed to close sslocal"
fi
```

可以将两个脚本的目录放到 PATH 里面，分别命名为`sss`和`close-ss`，这样就可以通过下面的命令启动和关闭shadowsocks的客户端了：

```
# 启用默认ipv4连接
sss 

# 启用ipv6连接
sss 6

# 关闭连接
close-ss
```

还是很方便的。
