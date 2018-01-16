---
title: 从头开始配置一个Fedora 24的双系统
date: 2016-07-18 14:35:20
categories:
- linux
tags: 
- Linux
---
大概两年前用过很长一段时间的fedora+win7的双系统，最早用的fedora版本还是14、15的样子，长期使用过18-20，后面几代都只在虚拟机里试玩了一下，因为配置起来一套顺手的桌面其实很麻烦，f24之后一直想再装一下fedora，在virtual box里面玩了一会觉得太卡就决定装双系统，花了两天时间基本弄得顺手了。

在今年，我的windows下的工具大多更换了，从印象笔记到为知笔记，同步使用了坚果云，音乐一直是网易云音乐，幸好这些工具都有linux版本的，装起来倒也不太费事，使用也还顺手。配置这个系统主要是一下几个方面：

- gnome shell theme. 就是gnome的主题，包括gtk主题，shell主题，图标等等，我现在使用 zukitre的gtk+主题， shadow的Icons， OSX-EICap的cursor， Zuki-shell的shell theme。这些都是在gnome-tweak-tool里面配置
- gnome-extension. 就是gnome-shell的扩展，这个非常重要，gnome的扩展性非常强，一些常用的扩展可以极大的方便操作，最常用的包括：applications menu, background logo, caffeine, coverflow alt-tab, dynamic top bar, launch new instance, media player indicator, openweather, place status indicator, simple dock, suspend button, topiconcs, user themes等
- 输入法，切换到fcitx和rime，除了sublime外基本都可以正常使用
- 工具类：网易云音乐，坚果云同步，为知笔记，WPS，mysqlworkbench， chrome，shadowsocks，vlc player，Atom, 甚至texstudio都有很好的支持
- 鼠标手势，在windows下习惯了 Wgesture的全局手势，一开在fedora下很不习惯，后来找到一个开源的 easystroke gesture recognition的软件，使用和Wgesture基本差不多，非常非常方便
- 使用Roxterm替换gnome的terminal，roxterm是我最喜欢的一个终端工具，定制之后非常方便

最后的效果：

![](/images/linux/fedora.jpg)

下面是安装时记录的一些问题，确实从头开始配置各种工具容易遇到非常多的问题，这里也是遇到一个记录一下，记得比较乱也不整理了，到时候再遇到能找到就可以了：
<!-- more -->
1. 必备工具： gnome-tweak-tool
2. infinality  https://danielrenninghoff.com/2015/11/22/infinality-ultimate-bundle-packaged-for-fedora/
3. http://www.lulinux.com/archives/278 开启infinality
4. gnome-look shell-theme: https://www.gnome-look.org/content/show.php/Zukitwo?content=140562
5. simple dock: https://extensions.gnome.org/extension/815/simple-dock/
6. shadow icons: https://www.gnome-look.org/p/1012532/
7. fix zsh command not found delay: `sudo vim /etc/PackageKit/CommandNotFound.conf` http://unix.stackexchange.com/questions/25681/why-a-long-delay-after-command-not-found
8. 使用U盘安装，需要使用win32diskimager 刻录到U 盘，否则会报错丢失文件。使用UEFI引导时则在磁盘分区的时候要求GTP模式创建 /boot/efi分区，以前的windows如果是MBR分区引导 则需要修改引导为Legacy模式。注意在引导时可能需要修改U 盘的盘符，默认的盘符名字太长会在写镜像的时候被windows截断，所以在引导时按 e 键，编辑LIVECD的label为正确的盘符。
9. 搜狗拼音： https://www.fdzh.org/blog/2015/10/18/fedora-sougoupinyin/
10. map caps-lock to escape: use `dconf-editor` edit `org.gnome.desktop.input-sources.xkb-options` > Add `caps:escape` to the aforementionned field, as : `['caps:escape']`
11. easystroke gesture 和windows下的wgesture功能很相似，https://sourceforge.net/projects/easystroke/  可以通过 `sudo dnf install easystroke` 安装
12. ultimate vimrc: https://github.com/amix/vimrc
13. fcitx: `sudo dnf install fcitx-pinyin im-chooser fcitx-configtool fcitx-qt5 fcitx-qt4`   https://wiki.archlinux.org/index.php/Fcitx_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)  , add 

```
export GTK_IM_MODULE=fcitx 
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

to `.xprofile`  http://www.dexcoder.com/selfly/article/4012 
14. fdzh repo: `dnf config-manager --add-repo=http://repo.fdzh.org/FZUG/FZUG.repo`  https://repo.fdzh.org/#
15. mysqlworkbench 经常奔溃可以尝试删除 `/home/<your username>/.mysql/workbench/wb_options.xml` 这个文件
16. everything you should know about gnome: https://wiki.archlinux.org/index.php/GNOME_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87) 
17. 如果 Gnome Shell 的界面卡死了（可能是由于某些外观调整异常、某个扩展出问题，或内存不足），你可能就连按下 Alt + F2 并输入 r 的机会都没有。这时，请试着切换到另一个 TTY（Ctrl + Alt + F2） 上，并输入命令pkill -HUP gnome-shell。Gnome Shell 将重新启动，可能需要个几十秒。这样的重启方式不会注销已登录的用户，因此所有的程序也将继续运行。不过，保存你正在编辑文件总是个好主意。如果这也不行，那你可能得重新启动 Xorg 服务器。如果你通过终端登录，输入 pkill X；如果你通过 GDM 登录，输入 systemctl restart gdm。但需要注意的是，重启 Xorg 服务器会导致已登录的用户被注销，因此请确保在这之前已经设法保存你所有的文件
18. texstudio下编译ieeconf的文件可能报错， `File ‘IEEEtran.cls' not found` 这是因为texlive有一些包没有装，可以 `sudo dnf install texlive-collection-publishers.noarch` 安装，但是这个包依赖特别多，如果只是解决这个问题，可以在 http://www.ctan.org/tex-archive/macros/latex/contrib/IEEEtran 这里下载 IEEEtean.zip 这个文件，解压出来把里面的 IEEEtran.cls 文件复制到你的 tex 文件所在的目录就可以了


