---
date: 2016/10/04 16:34:50
title: 一次LVM扩容记录
categories:
- server
tags:
- Liux
- LVM
- Fedora
---

之前写了一片重新装了一个Fedora 24的博客，那次装的时候为了系统速度比较快，把fedora装在了SSD上面，但我用的还是几年前买的三星的120GB的SSD，容量是捉襟见肘啊，只能从原来的windows系统分了35GB的空间给fedora，使用fedora自动分配后，挂载的可用空间只有29GB了，随着系统用的越来越久，剩余空间也就越来越少了，到现在只有10多G的可用空间了。幸好装Fedora的时候使用的LVM，这样可以很方便的进行扩容了。

## Shrink Windows Disk
首先从windows的分区压缩一些空间出来，当然LVM是支持从不同磁盘扩容的，但是为了速度还是从SSD中压缩空间吧，这样window留70G左右，剩余40G给Fedora，能压出来15G左右（因为影音类文件都放在另一个硬盘，所以45GB左右的Liux空间可以用比较长时间了）。
但是windows压缩空间有一个限制，那就是使用系统的压缩空间时，如果有一些不可移动文件占据了空间，那就只能从没有这类文件的位置开始压缩空间。。也就是说虽然C盘有30G的剩余空间，却不能压缩出30G，像我的情况就只能压缩出6G。。。这时候可以自己捣鼓也可以使用一个软件 AOMEI Partition Assistant 进行扩容，使用非常方便。但是！有一个问题，那就是需要重启进入一个XXX PreOS的界面自动重新分区，双系统重新分区后。。会导致Grub引导失效，只能进入 Grub Rescue 界面。

## 修复引导
还好Grub的修复还是有很多路子的，也许可以从Grub Recue中捣鼓，但是更方便的做一个Fedora Linux进入系统重新安装grub。
<!-- more -->
进入Live后，打开终端，使用 `fdisk -l` 查看Linux的引导分区是哪一个，主要看下面这个部分：

```
Device     Boot     Start       End   Sectors  Size Id Type
/dev/sda1  *         ****    ******    ******  350M  7 HPFS/NTFS/exFAT
/dev/sda2          ****** ********* ********* 60.7G  7 HPFS/NTFS/exFAT
/dev/sda3       ********* *********    ******  453M 27 Hidden NTFS WinRE
/dev/sda4       ********* ********* ********* 50.3G  f W95 Ext'd (LBA)
/dev/sda5       ********* *********  ******** 17.7G  7 HPFS/NTFS/exFAT
/dev/sda6       ********* *********   *******  500M 83 Linux
/dev/sda7       ********* *********  ******** 32.1G 8e Linux LVM
```

这是我的输出，找到Type为Linux的，我的是 /dev/sda6 ,二 /dev/sda5 就是刚刚从windows中压缩出来的空闲空间。找到这个引导分区后，重新安装grub引导如下：

```
su -
mkdir /mnt/root
mount /dev/mapper/Fedora-root /mnt/root
chroot /mnt/root
grub2-install --no-floopy --recheck /dev/sda
grub2-mkconfig -o /boot/grub/grub/cfg
```
这样应该就能更新grub引导了，双系统的话应该可以在输出看到 Windows 10  found之类的提示。

## 将新空间添加到LVM
重启进入系统后，就可以用LVM操作将空闲空间 sda5 加到LVM中了，之前又一篇讲LVM操作的： http://blog.wuxu92.com/Lvm-learning/ 大概也够用了，步骤如下：

### 使用sda5创建PV（physical volume）

```
sudo pvcreate /dev/sda5
sudo pvdisplay
```

### 添加PV到VG(volume group)

```
vgextend Fedora /dev/sda5  # fedora是我的vg名，可以用vgs查看系统的vg名
```

### 扩展lv（logic volume）
```
lvresize -l +100%FREE Fedora/root # fedora/root是我要扩容的lv，根据系统可能不同，用df可以查看
```

## 一个插曲
在使用 `lvresize`，使用 `-l`参数的`+100%FREE` 一不小心没有打那个 `+`号，这样就是把原来的lv大小变成了剩余空间的大小，我的情况是，100%FREE正好是刚刚 vgextends进来的 /dev/sda5 的17GB空间，这样原来32GB的 /Fedora/root 逻辑卷变成了17GB，这个误操作会报一个 WARNING因为调整后的LV变小了，可能会损坏系统的数据。。。但是一个Warning我也没有仔细看就继续执行了。。。

执行成功后，系统就卡死了 ，这时仔细看一下warning才发现有点大事不好了的感觉。。。强制重启后，果然进不了系统了，grub正常，但是引导到Fedora的图标界面后，进入一个shell界面，提示 `entering emergency mode ×××`，进不了系统了，能用的命令也非常少，基本没法抢救了。

这时候没办法只好又用Live系统，进入尝试将LV的空间扩大看能不能起作用了，使用命令如下：

```
su -
vgs # 查看vg列表
vgchange -a y Fedora    # identify and activate your vg
lvresize -l +100%FREE Fedora/root   # 这次正确执行将剩余空间加到LV

# 可以执行一下fsck
fsck /dev/Fedora/root

# 重启
sudo shutdown - r now
```

幸好，重启之后进入系统了，数据没有损坏，实在是幸运了。

## 扩展分区
上面的操作还只是将剩余空间加到了LV，这时候使用 `df -h`查看的话会发现，空间没有变化， `/dev/mapper/Fedora-root` 的空间并没有增大，我们还需要一步，将lv的空间加到文件系统中， xfs使用`xfs_grows`命令，我的是 `ext`文件系统，使用`resize2fs`命令，也比较简单：

```
sudo resize2fs /dev/mapper/Fedora-root
```

这个执行很快，一会就完成了，这时候再用 `df -h` 查看就能看到空间变大了。

完成。


参考：

- [https://help.1and1.com/servers-c37684/dedicated-server-Liux-c37687/administration-c37694/increase-the-size-of-the-logical-volume-a756168.html](https://help.1and1.com/servers-c37684/dedicated-server-linux-c37687/administration-c37694/increase-the-size-of-the-logical-volume-a756168.html)
- [http://blog.wuxu92.com/Lvm-learning/](http://blog.wuxu92.com/lvm-learning/) 
- [https://ask.Fedoraproject.org/en/question/34060/](https://ask.fedoraproject.org/en/question/34060/)


