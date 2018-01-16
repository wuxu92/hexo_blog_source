---
date: 2015/04/27 12:34:50
title: 使用ambari搭建并管理一个hadoop集群
categories:
- server
tags:
- bigdata
- hadoop
---

这部分包含的内容：

1. 什么是ambari
2. 如何安装Ambari
3. 安装hadoop集群的准备工作
4. 如何使用Ambari搭建一个集群
5. 其他

## 什么是Ambari ##
[Ambari](http://ambari.apache.org/)是的apache基金会下的一个开源项目。其目标是简化Hadoop集群的管理。通过Ambari提供工具可以方便的配置、管理、监控Hadoop集群。Ambari提供了一个直观的，易用的Hadoop管理web界面。Ambari现在最新的版本为2.0.0，但是网上比较多的资料都是1.2-1.7的。

作为一个开源项目，Hadoop在的配置与管理一直非常的不友好，要配置一个Hadoop集群经常会出现各种问题，并且对于集群的监控也比较麻烦，这也是像cloudera、hortonworks这类的提供Hadoop服务的公司可以存在的理由。

Ambari架构采用的是Server/Client的模式，主要组件：ambari-agent、ambari-server和ambari-web。ambari-agent是一个无状态的， 采集所在节点的信息并且汇总发心跳汇报给ambari-server， 处理ambari-server的执行请求。
<!-- more -->
需要注意的是**Ambari现在只支持64为的Linux系统，并且局限于RHEL 5/6，CentOS 5/6，OEL 5/6， SLES 11，Ubuntu 12**。这些是官方提供repository的系统，对于之外的系统需要自己编译安装。但是**对于32为的系统，不推荐去编译安装，因为我们在fedora 32bit上失败了**

## 安装Ambari ##
如果像我们一样使用CentOS 6.5系统的话，安装Ambari就非常简单了，参考[这里](https://cwiki.apache.org/confluence/display/AMBARI/Install+Ambari+2.0.0+from+Public+Repositories)。添加repo后直接

```
yum install ambari-server
```
就可以了。
安装完成后，需要对Ambari-server进行配置。

```
sudo ambari-server setup
```
这样会进入一个交互式的配置程序，主要配置项：

1. 关闭SELinux
2. 关闭iptables
3. 运行用户（默认root）
4. jdk配置，最好事先安装好最近的oracle jdk，否则Ambari会自己去下载安装jdk。选择自己配置jdk后，输入jdk的路径，默认是/usr/java/latest
5. 设置数据库，Ambari支持多种数据库，但是内置了postgreSQL，所以这一部分直接默认就可以了

大部分都默认就可以了，最好在配置之前就关掉SELinux和iptables，只要配置一下Java路径。
配置好之后，启动Ambari server

```
sudo ambari-serve start
```
### 安装Hadoop的准备工作 ###
要搭建一个集群当然首先要有几台PC，我们可以用虚拟机实现，推荐使用VirtualBox，比较方便，因为要集群的所有机器在同一个子网，需要使用桥接网络，VirtualBox的虚拟机的网络管理比较方便，kvm使用桥接网络有点麻烦。

首先配置好一台虚拟机，然后把这一台虚拟机复制几次就得到一个集群啦。不过**Ambari对配置要求比较高**，如果宿主机器的配置不太好就还是不要使用虚拟机了。

安装好虚拟机后，要配置虚拟机的ip，假设为192.168.100.1。现在虚拟机管理的设置里把虚拟机设置为桥接网络，只需要在网络选择的下拉框里选择桥接就可以了，一般不需要修改其他东西。

启动虚拟机后，配置一个固定ip的网络配置。然后需要修改机器的hostname（这也可以在安装虚拟机的时候设置好）。

```
hostname
hostname -f
```
这两个命令查看机器的hostname，他们的区别是，hostname -f使用的/etc/hosts文件中127.0.0.1 那一行的第一的名字，我们通过hostname命令修改主机名

```
sudo hostname 01.data
```
但是这样的修改会在虚拟机重启后被重置，需要修改一个网络配置文件。

```
sudo vim /etc/sysconfig/network
```
把文件中hostname相关的那行改为01.data。这样重启后，主机名也会是01.data

使用相同的方法，得到其他的虚拟机，分别将主机名设置为02.data,03.data,04.data,...并且为他们设好静态ip。

下面配置hosts文件。

```
sudo vim /etc/hosts
```
在文件中加上所有节点的ip与主机名的对应行，比如：

	01.data	192.168.100.1
	02.data	192.168.100.2
	03.data	192.168.100.3
	...

这样各个节点之间就可以通过主机名通信了。

下面配置ssh无密码登录
因为Ambari安装Hadoop集群时，要从Ambari-server所在的机器ssh登录到节点上面去安装Ambari-agent和Hadoop相关的组件，这需要Ambari-server到各个节点，以及各个几点到server之间的无密码登录，**各个节点之间不需要无密码登录**

** 最好设置root的无密码登录，因为我们配置的集群都是内网的，没什么安全性问题，使用root操作可以省去一些麻烦，非root用户可能在安装Hadoop组件时不能成功**

在server节点
1.切换到root用户，生成密钥对。

```
su
ssh-keygen -t rsa // 指定rsa算法,直接回车3次
```
2.复制公钥到节点

```
ssh-copy-id -i .ssh/id_ras.pub root@01.data
// 输入一次密码
```
3.验证无密码登录

```
ssh root@01.data
// 不需要密码就可以登录
```

4.重复上面的操作将公钥添加到所有的节点

然后因此远程到所有节点，同样的操作实现节点到server的无密码登录。

至此，准备工作就基本完成了，当然在实际操作中可能会遇到其他的问题，第一次操作可能比较费时间，尤其是节点多了，要挨个测试是很麻烦的。

## 使用Ambari搭建一个集群 ##
准备工作做好之后终于可以开始安装集群了。
在server端启动Ambari server，如果之前已经启动过了，可以用下面的命令停止或者重启

```
sudo ambari-server stop
sudo ambari-server restart
```
如果之前已经使用Ambari安装过集群，要删除原来的进度，可以运行reset命令

```
sudo ambari-server reset
```
，提示成功开启服务之后，就可以告别枯燥的命令行，通过浏览器操作了。
到开浏览器，访问 	`http://server-ip:8080`就可以访问ambari的管理页了。

后面的操作基本就比较简单了，按照顺序一步一步走就可以了，不过有几个我们遇到的问题在这里罗列一下：

1. 节点机器的ssh版本过低可能导致注册节点失败。这是航哥发现的奇怪的问题...
2. 推荐使用ssh private key的方式注册节点，那里需要添加的是ambari-server所在的机器的.ssh/id_rsa 文件，比较要注意的是，这里添加的是**私钥**
3. 刚开始可能有各种原因注册失败，根据提示一个个解决吧～
4. 注册成功后，可能会有一些warning，这些warning不想编程里面的warning，不要忽略他们，最好一个个都解决了，否则后面出了问题都找不到原因的。

列几个常见的：
1.thp没有关闭。hdp服务默认是开启的，这会导致Hadoop的资源占用还是什么的过高，推荐关闭，这个比较简单：

```
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
```
2.时间同步服务没有开启，

```
sudo service thp stop
sudo chkconfig thp off
```
3.磁盘空间不足。如果虚拟机使用的自动增长模式，很可能会导致这个问题，要求/目录下剩余容量不能小于2G，如果过少的，可以清理yum的缓存，或者删掉其他一些缓存、临时文件试试

4.各个节点之间通信的问题。第一次操作很容易碰到这个问题，主要主机名，hosts文件是否配置正确

后面的操作好像没有什么需要注意的了，安装完之后可能会有服务起不来的情况，这个第一次很容易碰到，具体原因很复杂，得自己分析报错的日志。

下面是我们遇到的一些奇怪的问题：

1. 节点的hosts文件配置了主机名到127.0.0.1。这会导致节点之间通信失败，其原因是这行hosts导致组件的端口监听绑定到了127.0.0.1而不是ip或者0.0.0.0，所以其他节点没法连接到这个节点。
2. 其他，不记得了。

## 其他 ##
搭建好之后，可以跑一跑基本程序，看看效果，Ambari的监控界面看起来还是很炫酷的。不过我们还没跑就是了。

总结：Ambari使用还是很有帮助的，虽然我们花了大概3天才搞通，不过，中间的问题真得自己去做才知道的，解决一个问题说不定得多长时间，写这篇烂文章都花了两个多小时呢。

最后，这篇文章就是抛个砖而已（砖没有加图，有时间再加）。
