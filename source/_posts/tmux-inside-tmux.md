---
title: 在tmux中嵌套一个远程tmux
date: 2018-03-03 14:31:43
categories: Linux
tags:
- Linux
---

在远端服务器执行某些很耗时的操作的时候，我们习惯使用tmux（或者screed）来保持远端的进程不因为ssh连接断开而终止。更复杂一点的情况，我们希望使用在tmux中连接一个远端服务器上的tmux会话，这在有些场景下会很方便。

首先描述一个这个场景，我们有一个开发机器A，这是一台局域网服务器，有root权限，用户独立，可以配置无密码登陆的。但是还有一个连接到现网服务的跳板机B，这是一台安全加强的服务器，无root权限并且严格限制用户权限，必须使用token登陆的服务器。每一次登陆B服务器都需要输入变化的Token作为密码，当然我们在B服务器上可以运行tmux会话复用终端，现在的场景是，希望在A服务器保持一个到B服务器的连接，以保证部分任务可以利用这个连接实现无密码在B 服务器上操作。

```
Local => A => B
```

直接的方法可以在一个tmux会话中ssh到服务器，然后使用ssh连接，然后在服务器 attach 到一个会话，但是这样有一个问题，tmux快捷键前缀复用的问题，一般我们习惯使用 `Ctrl+a`作为第二前缀，并且一般不怎么使用默认的 `Ctrl+b` 前缀。在上面的场景中，可以 SSH 到 A 服务器，然后在 A 服务器开启一个tmux会话，在会话中登陆到B服务器（输入一次Token），但是这样不能在这个连接终端复用，需要在这个到B 的连接开启一个Tmux会话，但是用两个问题要解决：

1. 上面说的tmux快捷键前缀操作符重复
2. 在现实界面有两个状态栏

第一个问题一般可以通过按两次前缀快捷键操作嵌套的B服务器的tmux会话，但是这样很麻烦，像老妈妈操作电脑一样，第二个问题其实比较好解决，可以通过tmux命令隐藏一个状态栏。

一直想怎么能比较“优雅”地解决第一个问题，但是都没有好的方法。之前同事推荐了一个github上面的配置，可以通过一个开关似的快捷键在A的tmux会话和B 的tmux会话之间切换，比按两次前缀好一些，但是要记住当前处于什么会话，也有点不好。

一直有一个想法是通过配置不同的前缀操作键（prefix）实现直接操作B端的会话。理解tmux的快捷键捕获的话这个思路就比较好理解，其实tmux是只捕获前缀按键（prefix）的，如果按下的快捷键不是前缀健就会发送给会话的连接。如果A的没有`Ctrl-a` 这个前缀，那么“寄居”在A的tux会话中的B 中的tmux会话就可以直接使用 `Ctrl+a`作为前缀操作了。

但是，有一个问题是tmux的前缀按键是在tmux服务中生效的。我们知道tmux能保持断开ssh连接继续执行，是因为tmux是一个BS架构的服务（tmux二进制文件本身既是服务器端有时客户端），后台运行一个tmux的服务，每次运行tmux，如果没有后台服务则启动一个服务，然后attach上去。如果后台已经有服务了则作为客户端连接到该服务。而prefix是服务启动时绑定的。也就是说prefix不是session级别的，所有的连接都一个服务的session都是一个前缀。

那是否可以通过开启多个tmux服务实现，之前捣鼓的时候尝试在一台服务器上运行两个版本的tmux服务，但是都失败了，如果已运行一个低版本的tmux，不能使用高版本的tmux作为客户端，并且创始使用高版本的tmux启动服务议会报`Lost connection`。但是对于同一个版本的tmux使用下面的命令可以启动两个服务：

```
tmux -L t2
```
但是tmux默认会找 `$HOME/.tmux.conf` 作为配置文件启动服务，所以会导致新的服务的prefix和之前的一样，所以我们要使用一个别的配置，或者直接使用一个空的配置文件，这样 `Ctrl-a`就不是新服务的prefix了。

```
tmux -L t2 -f /dev/null
```
这样，我们只需要在这个服务上创建会话，然后通过这个会话连接到B服务的tmux session就可以用 `Ctrl-a` 作为前缀了，这样**一个近乎透明的tmux嵌套**就实现了。几个关键的命令如下：

```
# 创建服务
tmux -L t2 -f /dev/null

# 连接到新的服务创建会话
tmux -L t2 [new-session -s jump]

# 在会话中连接到B服务器

# 在B中连接到新的会话
tmux attach session-name
```

这样可以用  `Ctrl-b`作为A上面的会话的前缀，用`Ctrl-a`作为B 的会话的前缀，一般来说不需要使用 `Ctrl-b`的。

可以在A服务器创建一个别名，更快捷地连接B的会话

```
# 在A 的 bashrc
alias sshb='tmux -L t2 attach -t jump'
```

当然可以关闭A上面的tmux的status-bar： `Ctrl+b :`进入tmux命令输入界面， `set status off`，就OK了。这样就像是直接连接到B服务器的tmux会话了，实际上是通过A服务器的tmux跳转一层的。
