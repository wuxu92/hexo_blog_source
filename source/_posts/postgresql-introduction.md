---
date: 2015/10/27 12:34:50
title: PostgreSQL 入门
categories:
- db
tags:
- database
- postgresql
- pg
---
![](/images/logo/postgresql.png)
## 安装 ##
被蓝屏搞没了
默认的initdb数据目录是 `/var/lib/pgsql/data/`

## 简介 ##
被蓝屏搞没了

## 入门 ##
> 重写

### 创建数据库 ###
pg的管理模式和mysql很不一样。在MySQL我们要登陆到mysql服务器才能进行创建数据库，添加用户等操作。但是pg提供了一个`createdb` 的命令，可以直接在bash中创建数据库。但是要求： 运行该命令的用户必须是启动postgresql服务的用户。

使用yum安装的pg会自动创建一个postgres用户并用这个用户启动相关的各个线程，所以要运行`createdb` 命令，需要先切换到postgres 用户。由于是系统创建的用户，我们也不知道用户的密码是什么，可以先重置 postgres 的密码。

```bash
sudo passwd postgres
// enter new password here

su postgres
createdb golang
// dropdb golang
```
当然也可以使用命令参数指定用户

```
createdb -U postgres golang
```
但是运行并没有成功，还是没有权限，切换到postgres用户比较好用。这样的管理方式对于习惯mysql的人来说，多少有些奇怪。

### 创建pg用户 ###
在登陆管理pg之前我们要学会怎么创建pg的用户，并用这个用户去登陆pg的服务。
说实话 [官方文档](http://www.postgresql.org/docs/9.2/interactive/auth-username-maps.html) 看起来可不是那么简单。
首先要切换到系统的postgres用户，登陆pg创建新用户。

```bash
su postgres
psql

postgres =# create user golang with login password 'golang'
postgres =# \q 
```
这样创建一个用户名为golang,登陆密码为golang的可登录用户。但是切换回普通用户，登陆还是会报错。

```bash
su wuxu
psql -U golang -W
Enter password
psql: FATAL:  Ident authentication failed for user "golang"
```
命名用户名和密码都是对的，却总是认证失败，注意前面的Ident authentication是什么意思？
这牵涉到pg的认证方式了。我们知道pg的认证方式是保存在配置文件 `pg_hba.conf` 中，这个文件在初始化数据(initdb) 所制定的目录，默认地址是 `/var/lib/pgsql/data/pg_hba.conf` ,这是一个比较标准的Liux风格的配置文件，一行代表一条规则。主要内容如下：

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
```
我们关心的是最后那个字段，METHOD，就是认证的方式，有很多种方式可以配置，常用的

- trust 任何连接都允许，不需要密码
- reject 拒绝符合条件(前面几个条件)的请求
- MD5 接收一个MD5加密过的密码
- password 接收一个密码来登陆，只在可信的网络使用这种方式
- gss 使用gssapi认证，只在tcp/ip连接可用
- sspi 只在windows可用的一种方式
- krb5 不常用，只在TCP/IP可用
- ident 使用操作系统用户名认证，验证它是否符合请求的的数据库用户名
- ldap 使用LDAP服务器认证
- cert 使用ssl客户端认证
- pam 使用操作系统的pam模块服务

根据上面的报错，是配置文件使用了ident认证方式，而用户golang并不是系统用户，所以会认证失败。所以我们要添加一行认证配置：

> 还有可能系统监听了 v4/v6两个端口，需要使用 `-h 127.0.0.1` 指定ipv4，否则会连接ipv6的端口，也会认证失败

```
host	golang	golang	0.0.0.0		password
```
然后重新加载配置文件 `sudo systemctl reload postgresql.service`.这样应该就能使用golang用户登录了。

### psql ###
psql是pg的中孤单交互程序，就像mysql的mysql程序一样。如下登陆： `psql -U golang -h 127.0.0.1 -p 5432 -W`，然后输入密码进入交互界面。

进入控制台有一些sql标准之外的命令，这些命令常用来管理pg。它们使用 `\` 开头，常用命令如下：

- \h：查看SQL命令的解释，比如\h select。
- \?：查看psql命令列表
- **\l：列出所有数据库**
- \c [database_name]：连接其他数据库
- **\d：列出当前数据库的所有表格**
- \d [table_name]：列出某一张表格的结构
- \du：列出所有用户
- \e：打开文本编辑器
- \conninfo：列出当前数据库和连接的信息
- \password 设置密码
- \q 退出

应该说pg的控制台比mysql的控制台人性化很多，特别是旧版本的mysql控制台真的很难用，不过5.6，或者mariadb 的控制台也有很大的改善了。

可以和mysql控制台一样在psql控制台运行各种标准sql语句。#注意字符串使用单引号# j建表，增删改查没有太大的区别。
官方给的创建表语句(我加了一个自增的字段,--表示注释)：

```
CREATE TABLE weather (
	id				SERIAL primary key,			-- same as auto_increments
    city            varchar(80),
    temp_lo         int,           -- low temperature
    temp_hi         int,           -- high temperature
    prcp            real,          -- precipitation
    date            date
);
CREATE TABLE cities (
    name            varchar(80),
    location        point
);
```
pg支持位置的point数据类型，它是一对数据，代表经纬度，这样插入数据可以如下：

```
insert into cities values ('beijing', '(100.0, 36.0)');
```
或者要从txt文件导入数据，在mysql中，我们使用load data local infile '' into table ** 这样的命令，pg使用copy命令导入：

```
copy table_name from '/path/to/file/of/server/'
```

一些对表结构的操作如下

```
# 添加栏位 
ALTER TABLE user_tbl ADD email VARCHAR(40);
# 更新结构 
ALTER TABLE user_tbl ALTER COLUMN signup_date SET NOT NULL;
# 更名栏位 
ALTER TABLE user_tbl RENAME COLUMN signup_date TO signup;
# 删除栏位 
ALTER TABLE user_tbl DROP COLUMN email;
# 表格更名 
ALTER TABLE user_tbl RENAME TO backup_tbl;
# 删除表格 
DROP TABLE IF EXISTS backup_tbl;
```
> 在生产环境中，不推荐使用 `select * ` 这样的查询语句，因为结果与表结构关联，表插入一栏后结果会改变。

### 联合查询 ###
多表联合查询实际上和mysql也没有太多区别，也就是标准的sql。

### 聚合函数 ###
Pg支持很多聚合函数，其使用和mysql并灭有太多区别。比如max.注意的是max不能用在where子句。

```
select max(tmp_lo) from weather;
select city from weather where tmp_lo = max(temp_lo);  // 错误
select city from weather where temp_lo = (select max(temp_lo) from weather);  // 正确
```
另一个常用的聚合就是group by了。没什么特殊的，和mysql同样的用法，注意group by 的条件子句使用 `having condition` 就可以了。

> WHERE selects input rows before groups and aggregates are computed, whereas HAVING selects group rows after groups and aggregates are computed. Thus, **the WHERE clause must not contain aggregate functions**; it makes no sense to try to use an aggregate to determine which rows will be inputs to the aggregates. On the other hand, **the HAVING clause always contains aggregate functions**. (Strictly speaking, you are allowed to write a HAVING clause that doesn't use aggregates, but it's seldom useful. The same condition could be used more efficiently at the WHERE stage.)

参考

- [http://www.postgresql.org/docs/9.2/static/tutorial-start.html](http://www.postgresql.org/docs/9.2/static/tutorial-start.html)
- [http://www.postgresql.org/docs/9.2/static/tutorial-sql.html](http://www.postgresql.org/docs/9.2/static/tutorial-sql.html)

done
