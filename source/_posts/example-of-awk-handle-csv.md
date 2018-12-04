---
date: 2015/08/19 12:34:50
title: 一个使用awk处理csv数据示例
categories:
- Liux
tags:
- Liux
- awk
---

最近要处理一个csv文件，起始是这样的：

|id | mmid | cp |
|---|------|----|
|zy006 | 2200112411 | 小米商城|
|zy007 | 2200123091-3003904564 | 掌游自有渠道 |
|zy008 | 0 | MM应用商城|
|zy101 | 2200131184-2200131784 | 多酷|
|zy102 | 2200017122-2200126498-2200127284-3003898651 | 3G门户|
|zy104 | 3003904473 | 软吧|

第二列是多个元素组合起来的，现在要导入到数据库里面，把第一列和第二列的元素使用一个关联表而不是这样使用数据拼接的方式。

要处理的结果是抽出第一列和第二列，然后第二列使用'-'分割新建一条记录。

步骤如下：

1 抽出数据到新文件:

```bash
awk -F "," 'NR>1{printf("%s,%s\n", $1, $2)}' data.csv> mmid.csv
```
NR代表当前的记录的数值，表示第几个记录了，这里忽略掉第一行。

2 分割数据并创建新记录

```bash
awk -F '[,-]' '{for(i=2;i<NF;i++) {printf("%s,%s\n", $1, $i)}}' mmid.csv > channel_mmid.csv
```
使用-F 设置两个分割参数，使用一个循环，其中NF是number of fields的简写，表名当前当有多少条记录，对应有NR，number of record

3 完成

使用awk处理excel/csv文件真的非常的方便

4 扩展

删除记录中的某一列

```bash
awk -F ',' 'BEGIN{OFS=","; } NR>1{str=""; for (i=1; i<=NF;i++) if (i != 2) str = str "," $i; print str}' channel-manage.csv > new-channel-manage.csv
```
