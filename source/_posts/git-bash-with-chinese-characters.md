---
date: 2015/12/18 12:34:50
title: Git Bash配置中文文件名、代理等
categories:
- tools
tags:
- git
---

![](/images/post/git.jpg)

Git默认的配置会将中文文件名显示为一堆数字（quote值），之前用也觉得没什么，文件管理的比较粗糙，乱码对应哪个文件都能猜出来，今天偶然看到这个配置，就水一篇好了。

其实只需要添加一个配置就可以了，

```
git config --global core.quotepath false
```
core.quotepath设为false的话，就不会对0×80以上的字符进行quote。中文显示正常。

水是水了点，确实还是有用的，尤其是对中文用户。

另外加一个配置Git自动管理Linux 和 Windows 的换行差别：

```
git config --global core.autocrlf true
```
这个配置在安装git windows 时是可以配置的，如果忘了配置可以用这个命令配置。

还可以配置下面的选项让git提交文件时不把换行符 warning 回显出来：

```
git config --global core.safecrlf false
```

**设置socks5代理**

```
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```