---
title: 有用的Bash命令记录
date: 2016-01-12 12:21:52
categories:
- linux
tags:
- bash
- linux
---

把一些有用的bash命令记录一下，一做学习，一做备忘。

## 文件与目录
**目录下批量文件重命名**

```
for f in *.jpg; do mv "$f" "${f%.jpg}"; done
for f in *; do mv "$f" "$f.jpg"; done
for f in $(find . -type f -not -name "*.*"); do mv "$f" "${f}.jpg"; done
```
这是遍历目录下的文件，修改为后缀 jpg，很方便，另一实现：

```
find . -type f -not -name "*.*" --exec mv "{}" "{}".jpg
find /path -type f -not -name "*.*" -print0 | xargs -0 rename 's/(.)$/$1.jpg/'
```
解压zip文件的特定目录

```
unzip <target-zip-file> '<folder-to-extract/*>' -d <destination-path> 
```
这会只对没有后缀的文件重命名，还可以根据需求做修改。

## iptables & firewalld
iptables其实还是很有规则的，只是很久不用就忘记了，今天配置新买的主机要打开一个端口，试了半天。。。

```
# 列出指定 rule chian 的规则
iptables -L INPUT
# 添加一条规则，其中的3表示插入作为表的第三条
iptables -I INPUT 3 -p tcp --dport 80 -j ACCEPT
# 虽然现在用systemctl 替代service的功能
# 但是保存iptables的更改还是只能用service
serbice iptables save
systemctl restart iptables
```

## 安卓
**去掉CM的感叹号**：这个东西真的有点烦，每次都要去搜，不如记一下：

```
# 使用 local shell 或者 adb
settings put global captive_portal_detection_enabled 0
# 修改验证服务器为 v2ex.com
adb shell "settings put global captive_portal_server v2ex.com"
```

**刷入 Recovery** 

```
# 刷入
./fastboot.exe flash recovery recovery.img
# 重启进入recovery
./fastboot.exe boot recovery.img
```

## 数据库
```
# 将一个数据库导入另一个库
mysqldump -u{username} -p{password｝ db_name | mmysql -u{username} -p{password} New_db_name
```

## 待续

参考：

- [http://stackoverflow.com/questions/1108527/recursively-add-file-extension-to-all-files](http://stackoverflow.com/questions/1108527/recursively-add-file-extension-to-all-files)