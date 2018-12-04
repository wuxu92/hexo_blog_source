---
title: 解决一个VirtualBox虚拟机错误导致的各种问题
date: 2016-04-13 21:07:47
categories:
- Liux
tags:
- VirtualBox
- Liux
- git
- tool
---

10.*.*.91是最早的一台虚拟机之一，是VirtualBox运行在Windows Server 2008 R2 上的，到现在已经运行了两年多了，基本没出什么问题。上面主要提供Git服务、一个Web server、一个Svn Server。前几天发现ssh登录的时候会直接返回 `connection failed`。并没有什么其他的信息。而直接ping则能ping通该IP。

非常奇怪的问题，由于前段时间一直没有用到这个机器就没管它。这几天重新做科委要提交git的时候总是失败（因为ssh都连不上，git肯定就提交不了了）。攒了很多commit在本地总觉得不安心，所以就找找原因，解决一下。

## 虚拟机停止运行
还好有Windows server的远程登录权限，登陆上去看了一下，91这台机器还在运行，但是试图从 Virtual Box 的图形界面登录时直接报错了，然后只能强制关闭该虚拟机，而运行的其它几台虚拟机都没有问题。仔细看了一下报错内容有一些 **DISK FULL**相关的内容。查看了一下 Virtual Box对该虚机的配置，32G的动态空间，已用17GB，没有问题啊，宿主机器（Windows Server 2008)分两块磁盘，C盘100GB，D盘750GB，而虚机使用的磁盘文件放在D盘，应该不可能磁盘空间不足的。
<!-- more -->
查看了一下宿主机器的磁盘空间，发现C盘剩余空间 0 KB....吓！什么东西把100GB的系统空间占完了，难道中了病毒？这机器只通过代理连接外网不应该中毒的，磁盘碎片整理了一下，还下了个CCleaner清理了一下，才多出来了几百M。。。。再运行91虚拟，果然运行成功了。

那就是C盘空间不足导致的了，找到用户文件夹下的Virtual Box文件夹，果然占用了40多G。。。原来**Virtual Box生成的快照文件默认会保存到这个目录**下（用户目录下的Virtual Box文件夹），怪不得。

把快照文件拷贝到D盘后，本想直接修改该虚拟机的快照目录，发现修改不起作用，点完确定之后就会被修改回原来的路径。。。。

## 从快照新建虚拟机
那就只能从快照新建虚拟机了，把快照文件的 vdi拷贝到D的目录下，新建虚拟机，然后选择已有的快照.vdi 文件（最好先重命名该文件，快照文件的文件名是一列散列码，太乱了）。我们知道每一个vdi文件都有一个uuid，Virtual Box下运行的虚拟机的磁盘文件的uuid不能重复，所以不能直接用快照文件，而需要为该 vdi文件生成一个新的uuid。执行下面的命令：

```
VBoxManage.exe internalcommands sethduuid "D:\path\to\old-file.vdi"
```
这个命令在Virtual Box安装目录下运行，默认位置为： `C:\Program Files\Oracle\VirtualBox`。

这样就可以成功使用快照vdi文件创建一个新的虚拟机运行实例了。而旧的虚拟机可以删除了。

## 新虚拟机的网络连接
新的虚拟机运行起来后，我从本地（个人计算机）能ping通10.*.*.91这个ip。但是ssh登录还是登录不了，浏览器也访问不了，使用 `ip a` 查看的ip也是91的，但是就是不能访问。

可以猜想91这台机器已经停机（崩溃）好几天了，91这个IP被释放然后被其他人占用了。。。学校的IP就是这点麻烦。换成自动获取后，获取到了一个 `10.0.2.15` 的IP，奇怪的是现在虚拟机能ping同其他机器了，虚拟机的浏览器都能上网了，但是本地机器还是连不到虚拟机。。。。在这个问题挣扎了好久，不停地换静态IP， network 和 NetworkManager 两个服务都快被我玩坏了，但是都不对，只有动态获取的ip能上网。。。

后来才想起来新建的虚拟机没有配置网络连接，默认连接方式为 NAT 。。。可不是外面连不到，关机切换到桥接（Bridge）方式，重启虚拟机，果然获得了一个 10.4 开头的IP，这和原来的IP是一个段的，但是静态配置到91的原来的ip就不行了，应该是ip冲突了，91的ip已经被占用了。

没有办法只好切换ip了，可惜这个用了两年多的91的ip。换ip的代价简直惨重，Git和Svn都得换ip了，本地配置的ssh登录也得更新，很麻烦但是也没有办法，91那个ip基本上抢不回来了。。。

## 配置Git Remote
首先更新gitlab配置的host域到新的ip，否则其他ip不能登录gitlab了，gitlab的配置在 `/var/opt/gitlab/gitlab-rails/etc/gitlab.yml`修改host配置就可以了，然后重启。

```
sudo gitlab-ctl restart
```
本地git仓库的remote url 也得更新：

```
# 查看原来的url
git remote -v show origin

# 设置url
git remote set-url origin git@new-ip:user/repo.git
```
这样就能提交了，只不过所有个原来的ip都要更新，很麻烦，应该在本地配个hosts的，那样就不需要改所有的ip了。


彩蛋：

> C盘剩余空间为0的时候，Windows Server 上运行的 phpMyadmin 的表结构都不能显示，而其他功能正常，在C盘恢复空间后就正常了。不知道两者之间是否有联系。

参考：

- [http://www.bradleyschacht.com/virtualbox-cannot-register-the-hard-drive-because-a-hard-drive-with-uuid-already-exists/](http://www.bradleyschacht.com/virtualbox-cannot-register-the-hard-drive-because-a-hard-drive-with-uuid-already-exists/)
- [http://www.ahLiux.com/start/base/21893.html](http://www.ahlinux.com/start/base/21893.html)
- [http://www.zhukun.net/archives/5949/comment-page-1](http://www.zhukun.net/archives/5949/comment-page-1)
- 