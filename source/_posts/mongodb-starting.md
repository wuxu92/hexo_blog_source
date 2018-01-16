---
title: MongoDB 学习
date: 2016-01-13 21:06:34
categories:
- db
tags:
- db
- nosql
- mongodb
---
很久之前（大概一年前）玩过一阵mongo，也就是安装了，curd什么的操作操作，到现在基本都忘记了，今天准备做个东西，想用mongo来存储，就顺便做点笔记吧。

还好之前安装的mongo的server还在，是3.0.7的，也能用了，就不升级了。

启动服务：

```
sudo systemctl start mongod
```
登录及常用操作，因为是本地服务也没有帐号密码直接就登录了：

```
mongod
use new_db
new_db.new_collection.insert({"name":"mi", "site":"baidu.com"})
```
这里，新建一个db并不要特殊的命令，只要 use 加新db的名字，然后再往里面插入一条数据，同样新的collection也直接使用就可以了，这样下来 `show dbs` 就能看到刚刚的db了。

```
use new_db	// change current database to new_db
show dbs	// list dbs
show collections	// show collections of current database
```


## golang 的 mongo驱动：mgo
mgo是Go的mongoDB 驱动，网站是： [https://labix.org/mgo](https://labix.org/mgo)， 文档比较可用但是也不是很全。

向要在项目内复用mgo的连接，结果实现很不好，似乎在 import 的时候会重新导入文件的变量，搞了半天都没有搞定复用连接

太二逼了，有一个底层调用用的旧的方法，所以总是新建了连接，我靠，花了好长时间才找出来，怪不得各种搜都搜不到内容

