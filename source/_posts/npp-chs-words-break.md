---
title: notepad++ 优化汉字换行
date: 2016-01-25 22:39:45
categories:
- tools
tags:
- tools
- windows
---

notepad++是一个非常好用的免费文本编辑器，用来替代Windows默认的编辑器非常的方便，但是在编辑时CJK字符会把一句话看作一个“单词”，导致换行显示的很奇怪，找了一下好像没有直接设置的选项，需要装一个插件来运行一个脚本（在每次启动时运行一次）

具体步骤如下：

1. Install NppExec (through the Plugin Manager is easiest)
2. In the menu go to, Plugins->NppExec->Execute...
Paste this text

```
sci_sendmsg SCI_SETWRAPMODE SC_WRAP_CHAR
npp_console 0
```
Click save and give it a name
3. Go to Plugins->NppExec->Advanced Options...
In the top right you can select a script to run at startup, select the script you just saved

This should set this option each time you start up N++