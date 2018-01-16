---
date: 2015/10/21 12:34:50
title: Linux的LVM相关学习
categories:
- server
tags:
- centos
- linux
- lvm
---
LVM 是一种可用在Linux内核的逻辑分卷管理器；可用于管理磁盘驱动器或其他类似的大容量存储设备。

## LVM的特点 ##
首先LVM有如下的优点：

- 文件系统可以跨多个磁盘，因此不会受物理磁盘大小的限制
- 可以在系统运行状态下扩展文件系统大小
- 可以增加新的磁盘到LVM的存储池中
- 调整逻辑卷大小的时候不需要考虑逻辑卷的位置，不需要担心没有足够的连续空间
- 可以以镜像的方式冗余重要数据到多个物理磁盘上
- 可以很方便的导出整个卷组并写到另一台机器上

当然LVM也存在一些限制：

- 卷组中一个磁盘损坏时，整个卷组都受影响
- 不能减小文件系统大小
- 存储性能会受到影响
- 从卷组中移除一个磁盘时必须使用reducevg

<!-- more -->
## LVM基本组成 ##
LVM利用Linux内核的device-mapper实现存储系统的虚拟化，使系统分区独立于底层硬件。通过LVM可以实现存储空间的抽象化，并在上面建立虚拟分区，可以极大地方便扩大和缩小分区。LVM的基本组成如下：

- 物理卷(Physical  volume,PV) 用于建立卷组的媒介，可以是硬盘分区，也可以是硬盘本身或回环文件。
- 卷组(Volume group,VG) 将一组物理卷收集为一个管理单元。卷组可以看成硬盘驱动器(hard drivers)
- 逻辑卷(Logic volume LV) 虚拟分区，由物理区域组成。逻辑卷可以看作普通的分区。
- 物理区域(Physical extent,PE) 硬盘可供指派给逻辑卷的最小单位，通常为4MB。

示例

两块物理硬盘的传统磁盘组织方式                
硬盘1 (/dev/sda):

     _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
    |分区1 50GB (物理卷)           |分区2 80GB (物理卷)            |
    |/dev/sda1                    |/dev/sda2                     |
    |_ _ _ _ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _ _ _ __|
                                  
硬盘2 (/dev/sdb):

     _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
    |分区1 120GB (物理卷)                         |
    |/dev/sdb1                                   |
    | _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _|

LVM方式
卷组VG1 (/dev/MyStorage/ = /dev/sda1 + /dev/sda2 + /dev/sdb1):

     _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ __ 
    |逻辑卷1 15GB                  |逻辑卷2 35GB                        |逻辑卷3 200GB                         |
    |/dev/MyStorage/rootvol        |/dev/MyStorage/homevol             |/dev/MyStorage/mediavol              |
    |_ _ _ _ _ _ _ _ _ _ _ _ _ _ __|_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ __ |_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _|

## 使用 ##
应该在系统安装过程中的分区和创建文件系统步骤创建LVM卷，而且根分区不再通知直接格式化硬盘来创建，而是创建在LVM逻辑卷上。

1. 创建物理卷所在的分区，设置分区的格式为Linux LVM。
2. 如果只有一个硬盘，最好只创建一个一个分区一个物理卷。如果有多个硬盘可以创建多个分区，在每个分区上分别创建一个物理卷
3. 创建卷组，并把所有物理卷加入卷组
4. 在卷组上创建逻辑卷

### 创建物理卷(PV) ###

```bash
# lvmdiskscan  // 扫描磁盘，列出设备
# pvcreate DEVICE 在列出的设备上创建物理卷 这里的设备一般是磁盘，比如/dev/sda2, 现在linux的一般使用sda1挂在/boot分区，把sda2挂载为lvm
# pvdisplay  查看建立的物理卷的信息
```

### 创建卷则(VG) ###
当有多个物理卷时，要把他们加入到一个卷组再在卷组建立逻辑分区。

```bash
# vgcreate vol_group_name <physical_vol>
# vgextend vol_group_name <physical_vol>  // 扩展lvm
# vgextend vgCentos00 /dev/sdb1
# vgdisplay //查看卷组的状态
```
也可以直接在命令中一步创建卷组和物理卷:

```
# vgcreate VolGroup00 /dev/sda2 /dev/sdb1 /dev/sdc
```

### 创建逻辑卷(LV) ###
在卷组上创建逻辑卷

```bash
# lvcreate -L <size> <vol_group> -n <logical_vol_name>
# lvcreate -L 100G VolGroup00 -n home
```
创建完后可以通过`/dev/mapper/VolGroup00-home` 访问分区

上面我们可以将多个磁盘加入到卷组中，如果我们要制定逻辑分区创建在某个磁盘上，比如把家目录放在速度较慢的机械硬盘上，只需要把物理卷设备加到命令中即可。

```bash
# lvcreate -L 100G -n home /dev/sdc
```

## 建立文件系统与挂在逻辑卷 ##
在逻辑卷上传见文件系统，并挂载

```bash
mkfs.ext4 /dev/mapper/VolGroup00-home
mount /dev/mapper/VolGroup00-home /home
```
注意挂载点要选择逻辑卷，而不是实际的分区设备(sda1, sdb1, etc.)

## 配置LVM ##
存在物理卷的设备在**扩增之后**或者**缩小其容量之前**，必须使用pvresize命令相应地增大或者减少物理卷的大小。执行命令 `sudo pvresize /dev/sda`,自动将物理卷扩展到其最大容量。

### 移动物理区域 ###
在移动空间的物理区域到卷尾部之前，需要运行 `# pvdisplay -v -m`查看物理设备的分段。这条命令会列出物理卷上的各个逻辑卷的分布。
具体的移动物理分区命令比较麻烦，请参考 [https://wiki.archlinux.org/index.php/LVM](https://wiki.archlinux.org/index.php/LVM "archlinux wiki") 中相关章节

### 调整逻辑分区大小 ###
可以使用lvresize命令对逻辑分区的大小进行调整，一般来说，扩大逻辑分区可以在线上完成，而缩小逻辑分区哦到小需要先卸载分区以避免数据丢失。

```bash
# lvresize -L +2G VolGroup00/home  // 增加2GB
# lvresize -L 20G VolGroup00/home  // 设置其大小为20GB
# lvresize -l +100%FREE VolGreoup/home  // 将所有空余空间都加入
```

使用`lvs`列出逻辑卷，使用`lsblk`列出逻辑卷的挂载点。

```
$ sudo lsblk
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda               8:0    0   128G  0 disk 
├─sda1            8:1    0   500M  0 part /boot
└─sda2            8:2    0 127.5G  0 part 
  ├─centos-root 253:0    0    50G  0 lvm  /
  ├─centos-swap 253:1    0   6.4G  0 lvm  [SWAP]
  └─centos-home 253:2    0  71.1G  0 lvm  /home
sr0              11:0    1  1024M  0 rom  

```
然后可以使用`unmount MOUNTPOINT`卸载逻辑卷

## 添加物理卷到卷组 ##
这在添加新磁盘的时候很有用。首先创建新的物理卷，然后把卷组扩展到该物理卷上。

```bash
# pvcreate /dev/sdd1
# vgextend VolGroup00 /dev/sdd1
```
可以使用`vgrerduce`命令移除VG中的PV。但是在移除之前要先试用`pvmove`把物理卷的数据转移到别的分区。

```bash
# vgreduce --all VolGroup00     // 删除所有物理卷
```

## 快照 ##
LVM可以给系统创建一个快照，由于使用了写入时复制(copy-on-write) 策略，相比传统的备份更有效率。 初始的快照只有关联到实际数据的inode的实体链接(hark-link)而已。只要实际的数据没有改变，快照就只会包含指向数据的inode的指针，而非数据本身。一旦你更改了快照对应的文件或目录，LVM就会自动拷贝相应的数据，包括快照所对应的旧数据的拷贝和你当前系统所对应的新数据的拷贝。这样的话，只要你修改的数据（包括原始的和快照的）不超过2G，你就可以只使用2G的空间对一个有35G数据的系统创建快照。

具体使用，没有经验，还是参考： [https://wiki.archlinux.org/index.php/LVM_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)](https://wiki.archlinux.org/index.php/LVM_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 吧

done

参考:

- [https://wiki.archlinux.org/index.php/LVM](https://wiki.archlinux.org/index.php/LVM"LVM在archlinux wiki上的页面")
- [http://icsmile.com/2013/04/22/lvm_1.html](http://icsmile.com/2013/04/22/lvm_1.html)
- https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)
