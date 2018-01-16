---
date: 2015/11/26 12:34:50
title: SELinux导致samba共享目录权限错误
categories:
- server
tags:
- samba
- linux
- selinux
---

之前将一台CentOS 7的机器使用samba共享家目录到Windows系统，方便使用。升级到Windows 10之后一直没有使用，昨天尝试连接samba怎么都连不上。

中间倒腾了很久，找了很多资料，改了很多配置，还是不行，总是报权限不足，请联系管理员什么的，如下图：

![](/images/post/samba-permission.png)

总结一下查到的几种可能存在的问题，虽然它们对我的情况好像都没有什么帮助：

1. windows 10会尝试使用samba 3_11链接，而现在samba server一般安装的是4.*的，所以需要disable掉Windows 10 samba 客户端的SMB 2/3协议，来自samba邮件列表 [https://lists.samba.org/archive/samba/2015-September/193886.html](https://lists.samba.org/archive/samba/2015-September/193886.html) ，禁用方法有微软官方方法 [https://support.microsoft.com/en-us/kb/2696547](https://support.microsoft.com/en-us/kb/2696547)。这个很重要
2. 对于无密码的samba共享，还需要设置一个注册表，找到 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters`，新建一个 `DWORD(32-bit)Value`，设置名字为AllowInsecureGuestAuth，值为1。有的人说这个有用。
3. 防火墙的问题，这个好解决，禁用掉iptables或者firewalld试试就知道了，要在iptables打开相关的端口，可以参考 [这篇文章](http://www.cyberciti.biz/faq/what-ports-need-to-be-open-for-samba-to-communicate-with-other-windowslinux-systems/)
4. selinux导致没有权限。这是我遇到的问题，我的server之前是setenfoce=0了的，后来机器断电重启就忘了这事了，结果昨天晚上倒腾好久都没想起来这茬，如果出现上面截图的情况请一定要

```
sudo setenfoce 0
```
这样就可以了。如果以前可以用现在不可以连接了，一般不会是samba配置的问题，考虑server防火墙和selinux比较靠谱。
