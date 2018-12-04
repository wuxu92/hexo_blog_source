---
date: 2015/08/13 12:34:50
title: 使用awk处理文本
categories:
- Liux
tags:
- Liux
- tool
- awk
---

在Liux下我们经常需要对一些文本文档做一些处理，尤其像从日志里提取一些数据，这是我们一般会用awk工具和sed工具去实现需求，这里对awk的入门使用简单记录。

awk可以看作一种文本处理工具，一种专注数据操作的编程语言，一个数据处理引擎。其名字来源于三个发明者的姓名首字母。一般在Liux下使用的awk是gawk(gnu awk)。

## 入门 ##
awk把文本文档看作是数据库，每一行看作一条数据库中的记录，可以指定数据列的分隔符，默认的分隔符是"\t",即Tab。
awk工作流程是这样的：读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域。默认域分隔符是"空白键" 或 "[tab]键"
awk的执行模式是： `awk '{pattern + action}' {filenames}`

awk的执行方式：

1.命令行方式

```
awk [-F  field-separator]  'commands'  input-file(s)
```
其中，commands 是真正awk命令，[-F域分隔符]是可选的。 input-file(s) 是待处理的文件。
在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。通常，在不指名-F域分隔符的情况下，默认的域分隔符是空格。

2.shell脚本方式
将所有的awk命令插入一个文件，并使awk程序可执行，然后awk命令解释器作为脚本的首行，一遍通过键入脚本名称来调用。
相当于shell脚本首行的：#!/bin/sh
可以换成：#!/bin/awk

3.将所有的awk命令插入一个单独文件，然后调用：

```
awk -f awk-script-file input-file(s)
```
其中，-f选项加载awk-script-file中的awk脚本，input-file(s)跟上面的是一样的。

一般是哟你哦个命令行模式就能满足需求了。

这样下面的一行文本：

```
abc def 123
dfg jik 234
```
在awk看来就是一个包含三个字段的记录，可以类比到mysql的一行记录，只不过awk没有一个mysql那么强的scheme。
这样比如我们要抽出中间的那一行数据，假设文本保存为文件 data.txt

```
awk '{print $2}'
```
很简单，这样就可以打印出中间的字符def 和jik 了。

下面来一个点点复杂的：

```
Beth	4.00	0
Dan	3.75	0
kathy	4.00	10
Mark	5.00	20
Mary	5.50	22
Susie	4.25	18
```
对于这样的数据
使用 ` awk '$3>0 { print $, $2 * $3 }' data.txt` 这样会输出

```
Kathy 40
Mark 100
Mary 121
Susie 76.5
```
理解就是可以在{}前面添加一个判断的语句，只有符合条件的行才会执行后面的语句。

## 进阶 ##
相对于print输出，可以使用printf进行格式化输出：

```
awk  -F ':'  '{printf("filename:%10s,linenumber:%s,columns:%s,linecontent:%s\n",FILENAME,NR,NF,$0)}' /etc/passwd
```
print函数的参数可以是变量、数值或者字符串。字符串必须用双引号引用，参数用逗号分隔。如果没有逗号，参数就串联在一起而无法区分。这里，逗号的作用与输出文件的分隔符的作用是一样的，只是后者是空格而已。

printf函数，其用法和c语言中printf基本相似,可以格式化字符串,输出复杂时，printf更加好用，代码更易懂。

