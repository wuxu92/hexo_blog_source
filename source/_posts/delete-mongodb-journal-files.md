---
date: 2015/11/13 12:34:50
title: MongoDB 删除journal文件
categories:
- db
tags:
- nosql
- mongodb
---

一台以前用来自己玩的虚拟机，只分配了6G的存储空间，今天进行系统更新的时候报磁盘空间不足。系统总共6G多一点点，主要空间都挂在在 `/` 下，由于很久没有更新系统了，这次更新需要下载300多M内容，在执行 `yum updata` 的时候就报磁盘空间不足了。

首先使用df看下空间 `df -h` 发现确实剩余空间不多了，但是这台机器只是我自己用来玩的，没有其他的东西，估计是记了很多日志吧，使用du看下是哪个目录占用比较多。

```
cd /
sudo du -d 1 -h
```
du命令的 `-d 1` 参数表示深度为1， `-h`使用用户友好的方式返回结果。使用了 -d 就不需要使用 -s 参数了。

查看发现 `/usr` 目录占用了4G多， `/home` 和 `/var` 目录倒都不算大。进usr目录在du看看。

```
cd /usr
du -d 1 -h
```
发现 `lib/mongdb` 目录占了很大的空间，再进去看，发现是一个journal文件夹占了3G左右。（如果对Linux比较熟的话，应该认识journal了，journal可以认为是一种日志记录方式，journal程序可以直接查询）

原来是很久以前玩mongodb的时候测试过插百万条数据什么的，估计就留下了这么大的文件。个人认为这是mongodb的事物日志记录，我现在不用mongo了，可以用下面的方法安全地删除掉。

```
sudo service mongod stop
sudo vim /etc/mongod.conf
```
插入一行配置 `smallfiles=true`
然后就可以删除那些journal文件了。

```
sudo rm /var/lib/mongo/journal/*
sudo service mongod start
```
这样就空闲了3G的空间了，自己练练手玩还是够的。

完
