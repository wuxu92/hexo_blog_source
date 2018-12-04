---
date: 2015/10/26 12:34:50
title: Go mysql数据库操作
categories:
- db
tags:
- go
- golang
- lang
- database
---

Go官方并不提供数据库驱动，而只是为开发数据库驱动定义了一些标准接口。我们要使用mysql数据库得使用第三方维护的mysql驱动。Go中支持MySQL的驱动目前比较多，有如下几种，有些是支持database/sql标准，而有些是采用了自己的实现接口,常用的有如下几种:

[https://github.com/go-sql-driver/MySQL](https://github.com/go-sql-driver/mysql) 支持database/sql，全部采用go写。
[https://github.com/ziutek/mymysql](https://github.com/ziutek/mymysql) 支持database/sql，也支持自定义的接口，全部采用go写。
[https://github.com/Philio/GoMySQL](https://github.com/Philio/GoMySQL) 不支持database/sql，自定义接口，全部采用go写。
也推荐使用第一个，主要理由：

- 维护的比较好
- 完全支持database/sql接口
- 支持keepalive，保持长连接

有了驱动，其实golang使用mysql和其他语言没有太大的区别，虽然具体的方法调用与类组织不太一样。

## 准备 ##
使用10.4.16.95上的mariadb数据库，新建一个golang的数据库，添加两张测试用的表。

```sql
create database golang;
use golang;

CREATE TABLE `userinfo` (
  `uid` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(64) DEFAULT NULL,
  PRIMARY KEY (`uid`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
CREATE TABLE `userdetail` (
  `uid` int(11) NOT NULL DEFAULT '0',
  `intro` varchar(128) DEFAULT NULL,
  `profile` text
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```
先ssh登录服务器(因为安全考虑，不开放mysql的root用户的远程登录权限)，用root用户本地登录mysql新建一个用于测试的可远程登录的用户：

```
ssh wuxu@10.4.16.95
MySQL -uroot -p
// enter root password
create user golang@'%' identified by 'golangMysql';
grant privileges on *.golang to golang@'%';
flush privileges;
```

这样就可以用golang登录新建的数据库了

## Golang操作mysql ##
先get数据库驱动,在命令行运行`go get github.com/go-sql-driver/MySQL`,会自动使用git从github下载mysql驱动工程到`$GOPATH/src/`。

基本的插入，更新，删除，查找方法如下：

```goalng
import (
	_ "github.com/go-sql-driver/MySQL"
	"database/sql"
	"fmt"
)

func main() {
	// mysql是一个注册过的数据库驱动了,在驱动的init函数中注册
	db, err := sql.Open("MySQL", "golang:golangMysql@tcp(10.4.16.95:3306)/golang?charset=utf8")
	CheckErr(err)
	fmt.Println("connected")
	
	stmt, err := db.Prepare("insert userinfo set username=?")
	CheckErr(err)
	
	// fmt.Println(stmt)
	res, err := stmt.Exec("wuxu")  // 使用占位符'?'防止sql注入，这是一个不定参数函数，传入的参数与prepare阶段设置的占位符相等
	CheckErr(err)
	
	id, err := res.LastInsertId()
	CheckErr(err)
	
	fmt.Println(id)
	
	stmt, _ = db.Prepare("update userinfo set username=? where username=?")
	
	res, err =  stmt.Exec("wuxu01", "wuxu")
	aff, err := res.RowsAffected()
	CheckErr(err)
	
	fmt.Println(aff)
	
	rows, err := db.Query("select * from userinfo")
	
	for rows.Next() {
		var id int
		var usename string
		err = rows.Scan(&id, &usename)
		CheckErr(err)
		
		fmt.Println(id, usename)
	}

	stmt, _ = db.Prepare("delete from userinfo where uid>?")
	res, _ = stmt.Exec(4)
	aff, _ = res.RowsAffected()
	fmt.Println("affected", aff)
}

func CheckErr(e error) {
	if e != nil {
		panic(e)
		
	}
}
```
与以往的mysql操作相似，也是一个连接对象，对于插入，删除等要先prepare，然后执行Exec，在exec阶段绑定占位符变量。
比较独特的是Exec返回的Result指针，它包含LastInsertId和 RowsAffedted 两个方法。

注意数据连接的dsn(data source name)和php或者其他语言的比较不一样。我们使用的是 `user:password@protocol(host:port)/dbname?charset=utf8` 的形式。驱动本身支持更多形式的dsn：

- user@Unix(/path/to/socket)/dbname?charset=utf8
- user:password@tcp(localhost:5555)/dbname?charset=utf8
- user:password@/dbname
- user:password@tcp([::]:80)/dbname   // ipv6

整个数据库操作的流程大概如下：

sql.Open()函数用来打开一个注册过的数据库驱动
db.Prepare()函数用来返回准备要执行的sql操作， 然后返回准备完毕的执行状态。
db.Query()函数用来直接执行Sql返回Rows结果。
stmt.Exec()函数用来执行stmt准备好的SQL语句。

数据库驱动的注册在数据库驱动包导入时执行，注册的逻辑写在包的init函数中。导入驱动包只需要 `import _ "driver/package"`，因为只需要执行其`init()`，而不需要调用包中导出的方法，调用的是go语言规定的sql接口。
