---
title: fstab自动挂载ntfs分区并设置权限
date: 2017-01-14 10:09:36
categories:
- linux
tags:
- linux
- fedora
---

由于是Windows和fedora的双系统，在fedora下需要挂载Windows下的几个分区，如果每次手动挂载实在太麻烦，可以在fstab里面配置自动分区信息，这样可以在开机的时候自动挂载这些分区了。

最开始的时候，使用下面的几行配置：

```
/dev/sda2 /run/media/wuxu/Windows ntfs defaults 1 2
/dev/sda2 /run/media/wuxu/bin ntfs defaults 1 2
```
但是这样挂载的分区使用默认的权限，也就是用户和组都是root，权限则是777，这在很多时候比较麻烦，比如一个带可执行权限的文本文件。我是配置的在打开带执行权限的文本文件时询问是否执行，实际上在Windows分区下的文件我只需要读写就可以了，这时可以如下配置权限：

```
/dev/sda2 /run/media/wuxu/bin ntfs defaults,uid=1000,gid=1000,dmask=022,fmask=111 1 2
```

其中uid和gid是你用户的id，可以通过`id -u`和`id -g`查看，dmask是目录权限掩码，注意是掩码而不是权限，真正的权限是 7减去掩码位，022对应的是755，即rwxr-xr-x。注意文件夹需要提供x权限才能进入。fmask是文件权限，配置为111取消可执行权限。

如上即可自动挂载并配置好Windows分区的权限了，也可以根据需要配置为其他权限。