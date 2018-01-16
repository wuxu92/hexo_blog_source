---
date: 2015/10/27 12:34:50
title: PostgreSQL 视图，外键与事物
categories:
- db
tags:
- database
- postgresql
- pg
---
![](/images/logo/postgresql.png)

## 视图 ##
视图提供了在真是数据表上建立抽象数据表的功能，可以封装数据表结构的细节，是一种很好的sql设计。实际中也会经常使用视图，使用基于视图的查询会方便很多。
创建视图的语法和mysql很像，我们知道mysql实现视图有两种算法，临时表方法和~~忘了另一种是什么了，它们的实现区别导致它们的性能有差异。在pg中创建视图似乎不需要考虑这么多，直接创建就可以了。

```
create view view_name as select subcluase
```
select 子句中还可以基于其它视图查询，基于视图创建视图也是很常见的。不过可能太多层的视图创建会对查询性能有影响。要设计好的表结构，尽量避免多层的视图依赖。

## 外键 ##
MySQL 的MyISAM引擎是不支持外键的，只有Innodb引擎可以支持外键。pg由于并没有多引擎的设计，所以在建表时不需要考虑这些，因为它就是支持外键的。

实际中，很多人并不太喜欢使用外键，比如我自己在设计数据库的时候也不喜欢用外键。一个是外键理解起来有点麻烦；二是外键的级联删除总觉得不太放心，不是自己操作的嘛。。。在使用外键的地方使用一张关联表也是很方便的。在插入删除的时候也就是多一句操作。

这里给一个官方的使用外键的例子，理解起来还是很方便的。

```
CREATE TABLE cities (
        city     varchar(80) primary key,
        location point
);

CREATE TABLE weather (
        city      varchar(80) references cities(city),
        temp_lo   int,
        temp_hi   int,
        prcp      real,
        date      date
);
```
也许使用外键是一个不错的习惯，以后的数据表应该学习使用外键。

## 事物 ##
事物使得一系列数据库操作打包成一个动作，保证中间不被打断以保证数据完整性。当其中一个操作出错时，系统会自动回滚到最初的状态，即这是一个 all-ot-nothing 的操作。
一个常见的食物操作的场景是转账或者付款交易。要保证A账号转出的数额与B账号受到的数额相等，不能A转出了B没有收到，也不能A还没转B就收到了。转账操作在数据库中至少需要两条语句，要保证这两天语句要么都成功，要么就都失败。
另外还要保证事物处理过程中，对外界是不可见的。

> The updates made so far by an open transaction are invisible to other transactions until the transaction completes, whereupon all the updates become visible simultaneously

pg中的事物非常的简单，只要是在 `BEGIN;` 和 `COMMIT;` 之间的操作就被看错一个事物。当然放到现实业务系统中，不同的驱动有不同的实现。
实际上可以把所有的语句都理解为事物，这是一个有意思的理解

> PostgreSQL actually treats every SQL statement as being executed within a transaction. If you do not issue a BEGIN command, then each individual statement has an implicit BEGIN and (if successful) COMMIT wrapped around it. A group of statements surrounded by BEGIN and COMMIT is sometimes called a transaction block

这里还有一个叫做 `savepoint` 的概念，在mysql中没有这个东西，pg的事务处理中，可以设置savepoint，以允许设置rollback的位置。使用 roolback to savepoint 可以保留savepoint之前的修改。一个示例：

```
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Bob';
-- oops ... forget that and use Wally's account
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Wally';
COMMIT;
```
## 窗口函数 ##

- avg
- rank
- sum

这部分和MySQL中很不一样，提供了更丰富的特性。吃饭去了，下次再写吧。
