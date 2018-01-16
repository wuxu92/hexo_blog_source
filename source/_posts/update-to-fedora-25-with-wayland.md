---
title: 更新到Fedora 25 with wayland
date: 2016-12-29 20:31:13
catgories:
- server
tags:
- linux
- fedora
---

毕业答辩完之后，时间也比较多了，前几天把很久没用的fedora upgrade到了 25。 Fedora 25是第一个默认用wayland替代了X的发行版，这个改动还是很大的，也是比较有风险的。

在F24中wayland是一个可选的显示服务，曾经尝试过几次，那个时候的wayland还非常不好用，桌面运行gnome会卡卡的，并且很多插件和软件都用不了。这次升级后的wayland可用性已经得到极大提升，虽然还有一些小问题，但是日常使用已经没有问题了。

Fedora官方给出了方便的[升级方法](https://fedoraproject.org/wiki/DNF_system_upgrade)，推荐使用 dnf plugin 方式在线升级，核心方法就4条命令：

```
sudo dnf upgrade --refresh
sudo dnf install dnf-plugin-system-upgrade
sudo dnf system-upgrade download --refresh --releasever=25
sudo dnf system-upgrade reboot
```
更新和dnf update差不多，不过会在重启的时候安装更新，我的系统大概有7000个更新，花了半个小时的样子，非常顺利更新完之后重启，进入新系统。

更新的系统不会损失原来的文件和配置，gnome的插件也都还在，使用terminal的时候，菜单有一点点卡顿，使用一段时间后不太明显了。大部分的软件都正常运行。

几个运行异常的地方：

1. 原来配置好的 QT5 输入法问题，升级后在为知笔记不能使用搜狗输入法，其他软件的输入法没有问题。
2. 显示器自动色温调节软件，f.lux和redshift都不能运行，因为色温调节依赖X提供的接口，wayland还没有适配
3. 下拉的关机按钮还是有可能导致系统卡顿，这在x做服务的时候就存在，应该是gnome的bug

不知道是不是更新到了wayland的错觉，整体感觉更流畅了，到现在也没有崩溃的情况，值得更新到Fedora 25。

随手截个图看下效果：
![fedora screensnap](/images/movie/fedora-25.jpg)

