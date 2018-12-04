---
date: 2015/10/19 12:34:50
title: 重置mysql的密码/无密码登录mysql
categories:
- db
tags:
- MySQL
- Liux
---
经常会忘记mysql的root密码，尤其是机器一多的时候。上周五把科委的服务器迁移到centos上面，当时把mysql配置好了，也能登录了，但是今天各省测试的时候，发现mysql链接失败，登录服务器用命令行登录发现是 "Access Denied",明明记得是正确的密码，又不能登录了，估计又记错了。

经常会忘记这些密码，干脆记录一下怎么样重置密码吧。我们知道mysql的用户信息是保存在mysql的一张表里面的，如果要重置密码则需要登录到mysql的服务器。按照mysql的官方文档，可以使用不需要密码的方式启动mysql服务器，这样就能不需要密码登录了。

先关闭mysql

```
sudo service mysqld stop
```
然后启动无权限表限制的服务

```
sudo mysqld_safe --skip-grant-tables
```

登录mysql, 设置密码

```
MySQL
set password for 'root'@'localhost' = PASSWORD('yourPassword');
```
注意设置密码需要使用PASSWORD函数处理，也就是把密码hash存储。

重新启动正常的服务

```
sudo kill `cat /var/run/mysqld/mysqld.pid`  // 也可以通过ps查找pid再kill
sudo service mysqld start
```
当然，其他账号同理可以修改密码了。注意不要开启root的远程登录权限。
done

参考: [https://dev.MySQL.com/doc/refman/5.0/en/resetting-permissions.html](https://dev.mysql.com/doc/refman/5.0/en/resetting-permissions.html)
