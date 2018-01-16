---
title: Linux将CapsLock映射为额外的Escape
date: 2017-01-25 12:28:14
categories:
- linux
tags:
- linux
- fedora
---
大写锁定键是一个没有什么用的按键，一般需要大写输入习惯使用 Shift加字母实现，很多时候一不小心开了大写锁定没有发现还会导致输入错误，所以我的机器的大写锁定都是映射到Esc键的，习惯了之后会发现这是一个非常有用的设置，顺手，能提高效率。

在windows下可以用软件写注册表实现，并且操作一次后就不需要管了，一直工作很好，但在我的Fedora上用之前的方法，一种是通过gnome-tweak-took的type分类里面有一个设置项可以设置capslock的功能，另一种方式是使用dconf-editor方式，设置 `org.gnome.desktop.input-sources xkb-options`的值为 `['caps:escape']`。这两种方式的底层实现应该是同一种方式，在升级到 fedora 25之后，这种设置偶尔会失效，打开一次 gnome-tweak-tool就会正常一段时间，然后又失效了。
<!-- more -->
还有另一种方式可以设置按键的动作，那就是使用 Xmodmap， 之前也了解过但没有使用过，查了一些资料，发现其实也不麻烦，特别是在fcitx输入法下，通过其设置可以很方便，如果熟悉 xmodmap 可以实现很多按键功能。具体步骤如下：

1） 编辑一个 xmodmap 文件，比如新建 `~/.Xmodmap`，输入如下代码：
```
clear Lock
keysym Caps_Lock = Escape
add Lock = Caps_Lock
```
一般来说，家目录下的 .Xmodmap 文件会被 xmodmap程序自动执行，也就是有这个文件上面的映射可以自动生效，但是也可以在 fcitx的配置中在配置一下。
2） 打开 fcitx-config 程序，在 Addon 选项卡下，勾选 Advance 选单，找到 X Keyboard Integration 双击设置xmodmap的脚本文件为刚刚添加的文件即可。

通过上面的方法，不需要重启，CapsLock按键就作为一个额外的Escape了。