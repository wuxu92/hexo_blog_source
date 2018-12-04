---
title: 解决新Fedora下vboxdrv不能加载的问题
date: 2017-12-03 22:21:02
categories:
- Linux
tags:
- Fedora
- Linux
---
> 这是一篇旧的笔记，最近Append了一些内容，发出来可能对有的人有用

Fedora在7月发布了版本号为26的新版本，在上周把家里的笔记本先升级到26踩坑，基本上平滑升级，没有太多问题，gnome的插件也兼容的比较好，除了几个不太常用的基本没有broken的。

可惜的是升级之后VirtualBox不能用，报的是比较常见的kernel driver not installed 的错误，以往按照给的提示操作一下安装好kernel driver就没问题了的，这次折腾了好久也不行。期间尝试过：

1. 更新/重装 virtualbox
2. 换到Oracle的源安装VirtualBox
3. 手动运行 `/sbin/vboxconfig`，编译kernel driver
4. 禁用modprobe `kvm_intel` [Ref](http://www.Liuxquestions.org/questions/linux-newbie-8/modprobe-vboxdrv-error-i%27m-stuck-for-3days-4175493638/)  再重装
5.安装旧版本内核后重装virtaulbox

毫无意外以上的方法都失败了，经过一顿折腾感觉问题应该处在 secure boot上面， 想进BIOS把secure boot关掉，但是发现笔记本的BIOS密码忘记了，这条路也走不通了，不过我感觉就算关掉secure boot应该也不行。

后来找到[一篇文章](https://gorka.eguileor.com/vbox-vmware-in-secureboot-Liux/  )，自己对这个mod签名"骗过"操作系统，其实之前也看到了这篇文章，但是觉得有点麻烦就没怎么细看，今天研究了一下，其实也挺好处理的，就是生成一个key，然后用这个key给未签名的module签名，再通过mokutil导入key，重启系统就可以了，关键命令记录如下，具体细节可以查看原文。

```
sudo modprobe -v vboxdrv        # 查看mod信息，在这可以看到报错信息
# 生成key， 包括der和priv文件
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=Akrog/"

# 使用生成的key为mod签名
sudo /usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vboxdrv)
sudo mokutil --import MOK.der    # 导入，注意这里要用sudo执行，前面可以用普通用户执行
# 这里会提示输入密码，这个是重启时导入证书需要输入的密码

sudo reboot        # 重启进入导入证书的界面，按照操作即可
dmesg | grep 'EFI: Loaded cert'    # 启动后查看我们刚刚生成的证书已经加载

# 重启后vboxdrv会自动加载，直接启动virtualbox即可。
```

VirtualBox可能会再运行时导致Host的CPU利用率飙升至100%，可能的原因是开启了vb的nested paging设置，再设置里面关掉可能就好了，该项设置可能需要先关闭虚拟机，设置路径在： `Setting - System - Acceleration`。go to the system tab, click on Acceleration and then uncheck the Enable Nested Paging checkbox.  Click OK and start the virtual machine up and you should quickly notice some performance improvements 参考： [DotNetMafia](http://www.dotnetmafia.com/blogs/dotnettipoftheday/archive/2010/09/22/fix-high-guest-cpu-utilization-in-virtualbox-by-disabling-nested-paging.aspx)  
和有一个CPU 飙升到100%的问题可能时虚拟机内某些程序有bug，比如我常用的为知笔记的后台进程就经常发疯到100%，在windows task manager 中kill掉就好了。

APPEND:

每次升级内核会出现模块签名无效的问题，所以升级内核需要使用对应新内核的 sign-file 脚本执行一遍，可能除了 `vboxdrv`，还有其他模块需要签名的，这些需要签名的列表可以在 执行 `sudo systemctl restart systemd-modules-load.service` 之后（这时通常会失败）,通过 `journalctl -xe` 命令查看，只需要找出加在失败的模块然后使用之前使用过的 MOK.priv 和 MOK.der 两个文件签名一下就可以了，如果旧的密钥文件丢失了，就重新走一遍上面的流程，如果有旧的MOK文件则不需要 import MOK.der 的阶段了，比如升级 4.13.16内核之后就需要重新签名好几个module：

```
sudo /usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vboxnetflt)
sudo /usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vboxnetadp)
sudo /usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vboxnetpci)
sudo /usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vboxpci)   
sudo systemctl restart systemd-modules-load.service
```

当然也可以禁用内核的升级省得麻烦，毕竟个人系统对内核版本要要求也没有那么严格，只要在 `/etc/dnf/dnf.conf`中加入 `exclude=kernel*` 这样的一行就会在 `dnf update`的时候排除kernel的更新了。

