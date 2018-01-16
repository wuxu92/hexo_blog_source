---
title: snp中MongoDB设计失误
date: 2016-01-30 12:18:46
categories:
- db
tags:
- mongodb
- go
- snp
- db
---
snp是前段时间开始的一个[个人项目](https://github.com/wuxu92/snp)，受Chrome导航插件 FVD SpeedDial的启发，做一个自定义的导航页，后面会陆续写一些这个项目的文档，今天主要是总结一下到现在为止，MongoDB设计的一些失误。

snp使用MongoDB作为数据存储，MongoDB作为一个老牌的NoSQL数据库，比较适合这个项目的需求，之前一直是使用MySQL做数据库，虽然MySQL肯定也能满足需求，但是一来为了熟练一些MongoDB的使用，而来NoSQL存储对应的数据确实更加方便，就没有选用MySQL了；一开始设计时还考虑过使用SQL Lite做存储，因为SQL Lite足够小，部署比较方便，不过snp本来是一个练手的项目，学习MongoDB也是其中目的，也就没有使用SQL Lite了。

## 数据描述
我们知道NoSQL的一个很大的优势是没有了关系型数据库的模式约束，可以在一个数据集合中快速存取不同“schema”的数据。
<!-- more -->
先介绍一下snp中主要使用的数据，如果使用关系型数据库描述，它们主要包括：

### package数据
| 字段 | 类型 | 备注 |
| -- | -- | -- |
| Id  | string | ObjectID |
| Created | string | 创建事件 |
| Title | string | 标题 |
| Owner | string | 创建人 |
| Name  | string | 名字 |
| Password  | string | 密码 |
| Public  | bool | 是否公开 |
| Groups  | []ObjectID | 组的 ObjectID 数据 |
| Theme | string | 主题 |
| AdditionalStyle | string | 附件样式表 |
| Version | int | 版本 |

应为使用MongoDB，id就是不是MySQL的自增主键了，而是MongoDB自动生产的hash过的id字符串，上表中需要注意的是保存Groups的数组，我们知道NoSQL可以在一个字段存储数组，对应于MongoDB则是类似于JavaScript数组一样的数据类型。

这里为什么不会直接存储Group的对象数组而是存储Group的id的数组呢？这是因为snp的设计中Group是可以包间共享的，也就是follow的概念，所以这里是使用ID的数组。

我现在觉得设计失误的是组和网站的数据集合，由于常年使用关系型数据库（MySQL）的影响，在设计时使用就得思维方式，将Site单独抽象出来了一个集合。这其实是没有必要的，虽然一开始考虑过Site在Group之间的共享，但是后来发现这样是没有意义的，因为不同Group添加的网址有自己的一些属性，如添加时间，特别是组可能会编辑网址，这样如果网址共享的话，会导致一个地方修改，所有人的导航页都被修改了，这显然是不合理的。另一个方案是，可以借鉴 写时复制，在编辑时对Site做一份拷贝，这样可以减少系统中重复网址的开销，不过这样实现有些麻烦，并且省下来的那些空间并没有多少，反而导致过多的计算。

下面看一下现在的Group和Site设计

## Group和Site

Group数据

| 字段 | 类型 | 备注 |
| -- | -- | -- |
| Id | string | ObjectID |
| Created | string | 同上 |
| Title | string | 同上 |
| Owner | string | 同上 |
| GroupLink | []ObjectID | 备用 |
| Followable | bool | 是否可Follow |
| Localized | bool | 是否本地 |
| FollowFrom | ObjectID | 从某Follow |
| AdditionalStyle | string | 同上 |
| Sites | []ObjectID | 包含的网址 |
| Version | int | 版本 |

Site数据

| 字段 | 类型 | 备注 |
| -- | -- | -- |
| Id | string | ObjectID |
| Title | string | 同上 |
| Url | string | 链接 |
| Created | string | 同上 |
| Available | bool | 备用 |

可以看到这里设计的失误，Group中不应该存储Site的ID数组，而应该直接存储Site的对象数组，因为Site实际就是Group的成员。现在这样的设计导致在查找Group的所有网站时，需要进行更多次的数据库查找，在查找package的所有网站时导致查询次数成倍增加！！

但是现在已经写了不少的接口，再进行修改需要重写那些功能，实在是麻烦，只能将错就错了。。。
