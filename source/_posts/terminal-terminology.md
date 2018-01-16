---
title: 一个不错的终端工具：terminology
date: 2017-06-23 11:59:06
categories:
- linux
tags:
- linux
---

之前一直使用gnome自带的terminal，感觉也挺好用的，就是多开tab的时候有点丑，习惯了其实也没事，但是偶尔就是想换掉它。。更早的时候使用的 roxterm也是很好用的，但是不知道为什么从F23开始，border样式变得很奇怪，有一圈没法去掉的大边界。

找了好几次terminal替代品，昨天试了好几个，有 st, roxterm, terminology, ternimator,xfce terminal, guake 等，都各有好处，但是没有一个完全满意的。。。要是有早期的roxterm的效果也就很满意了。

其中有一个 terminology 挺有特色的，有很多华丽的效果，简直duang duang的，但是有几个问题，一个是配色调起来很诡异，一个是远程的tmux缩放窗口不能自动调节，多tab倒是挺好看的，非常适合我的土鳖审美，经过一顿调教，总算比较好用了。

1. 首先配色选 solarized， 但是这时文件夹和文件的配色很难分清楚，还得在 color项目下，重新选一下配色，并勾上 use
2. 字体得慢慢调，粗看起来会比较糙，字体和大小都得慢慢调，现在用 droid sans mono(Regular) 14号字，感觉还可以
3. 针对tmux的配色，需要在 behavior 选项勾选 set TERM to xterm-256color， 否则配色会很糟糕，勾选后似乎重启才能生效，关掉ring bell也在这里面
4. 针对tmux自动缩放的问题，似乎需要kill session重新启一个session才正常，这个有点麻烦，不过弄好了就很舒服了
5. 终端透明在video中调一下就ok了
6. 还可以设置终端的壁纸，这功能看个人需要，感觉设置了有点非主流，还是朴素一点比较好
7. 快捷键的设置很方便，也算是一个优势

terminology还有样式看起来很舒服的分栏功能，比绝大部分终端软件的分栏看起来都舒服，但是习惯了tmux的分栏还是不推荐再使用terminal的分栏功能，太弱了，并且容易搞混。

一个截图：
![terminology screenshot](/images/linux/terminology.jpg)

**一个致命的缺点**： 不能使用fcitx-rime输入法，不能输入中文。测试了一下，ibus和fcitx都不能输入中文，如果有中文输入的需求。。。还是暂时不要用这个了，用gnome terminal吧。

**又一个致命缺点** ：快捷键冲突，terminology本身有还多快捷键，没有快捷的禁用其快捷键的方法，而terminal本身的快捷键容易覆盖掉程序的快捷键，如vim和tmux的。发现没有办法在vim的panel之间快速跳转了，所以我放弃这个terminal了。
