---
title: 服务器ssh登录保护
date: 2016-01-27 17:43:33
categories:
- server
tags:
- Liux
- vps
- ssh
- tools
---

今天登录前段时间买的VPS，发现motd消息中有一段如下：

```
There were 405 failed login attempts since the last successful login
```
基本过几个小时再登录就有这个消息，小的几百次失败登录尝试，多个可能几千次，看来网络上真的是错综复杂，不安全呐，整天有人想着办法要“勾搭”互联网上的server。没办法只好配置一下SSHD，要不这么一天到晚被尝试登录也受不了。

如果是购买的远程主机建议配置到普通用户的密钥登录，关闭root登录权限和密码登录权限，最好把sshd的端口改了，因为大部分尝试登录都是针对默认端口22的。下面是操作步骤。

因为是远程主机所以操作要按照一定的顺序，要不然可能导致不能登录，注意的是在配置过程中不要退出已登录的回话，直到配置完再退出最初的会话。

## 创建新用户
用现在的用户名密码登录到服务器，这个session不要关闭，中间对sshd的配置及sshd服务重启不会断开这个连接的。

```
useradd me
passwd me
usemod -aG wheel me
```
这里假设已经编辑 visudo 将wheel用户组设置为拥有 sudo 权限。如果还没有的话使用 `sudo visudo` 找到相应的行取消注释即可，注意千万不要手动编辑 `/etc/sudoers` 文件，以前同学手动编辑该文件导致的惨烈后果记忆犹新。。。。

## 配置无密码登录
我们都知道要实现无密码登录就是在本地生成一个密钥对，然后使用用户名密码将公钥(.pub)上传到服务器，然后登陆时指定对应的私钥，就不需要输入密码了。

### 生产密钥对
使用下面的命令

```
ssh-keygen -t rsa -f me.server
```
<!-- more -->

-t表示密钥使用的算法， -f是生成的密钥对的文件名，默认回事id_rsa，不过为了和其他服务器的密钥对区分，建议使用一个直观的名字，而不是默认名字，上面的指令会询问是否使用密码，这个密码是密钥对使用的密码，建议使用一个简单的密码，这样比没有密码安全很多，也不会有额外的记忆负担

```
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```
然后回车后就会在 `~/.ssh/` 目录下生产两个文件，分别是 `me.server` 和 `me.server.pub` 它们分别是私钥和公钥文件。

### 上传公钥到服务器
很多人喜欢使用手动上传文件拷贝内容到服务器的 `~/.ssh/authorized_keys` 文件中，这样很麻烦也容易出错，其实有一个非常方便的工具 `ssh-copy-id` 命令专门用来上传公钥，使用可以查看帮助，一般如下：

```
ssh-copy-id -i ./me.server.pub -p your_port me@host
```
使用 `-i` 指定公钥文件，否则默认使用 id_rsa.pub 就不对了，是哟个 `-p` 指定sshd的端口，现在我们还没有修改服务端的端口就使用默认端口22或者不写就可以了，最后是用户名和服务器地址。

这个指令会询问登录密码，输入密码就可以了，指令会自动将公钥上传到服务器，并将内容追加到指定用户的 `~/.ssh/authorized_keys` 文件中，你可以在服务器中查看该文件

### 使用私钥登录
在ssh登录时指定私钥文件可以配置 `~/.ssh/config` 文件指定某个域名的私钥，或者在ssh命令指定私钥文件。ssh/config 配置格式如下：

```
host alias
        User me
        Hostname host
        Port 22
        IdentityFile ~/.ssh/me.server
```
这样配置的好处是可以为连接自定义一个别名，在连接时指定这个别名如： `ssh alias` 就可以了。

或许使用 `-i` 参数指定密钥文件：

```
ssh -i ~/.ssh/me.server -p 22 me@host
```

## 修改sshd服务配置
sshd的配置文件是 `/etc/ssh/sshd_config`,主要做下面几个配置修改

1. 禁止root用户登录， 找到`PermitRootLogin`配置，取消注释，将其值修改为 `no`
2. 修改登录端口，在配置文件中寻找 `Port` 修改为另一的端口，注意不要与其它常用端口冲突，可以设置到10000以上的端口
3. 禁止密码登录，完全禁止使用密码登录，这样那些尝试暴力登录的工具就完全没有办法了，找到 `PasswordAuthentication` 的配置项，将其值修改为 `no`

保存退出后，重新启动服务： `sudo systemctl restart sshd` 或者 `sudo service sshd restart`


注意：
1. 修改了默认登录端口后记得修改客户端 ssh 的config文件中配置的端口要相应修改
2. 修改了端口之后一定要记得打开防火前对应的端口，否则以后就登录不了了：

```
sudo iptables -L
sudo iptables -I INPUT -p tcp --dport 12222 -j ACCEPT
sudo service iptables save
sudo systemctl restart iptables
```
这样更新iptables的规则后就可以指定新端口登录了。

Windows下使用 Xshell 可以保存指定 rsa 私钥和passphrase，登录也很方便。

完。

参考:

- [https://www.Liux.com/learn/tutorials/843903-how-to-make-your-linux-server-more-secure%20to%20secure%20your%20server](https://www.linux.com/learn/tutorials/843903-how-to-make-your-linux-server-more-secure%20to%20secure%20your%20server)
- [http://www.cyberciti.biz/faq/force-ssh-client-to-use-given-private-key-identity-file/](http://www.cyberciti.biz/faq/force-ssh-client-to-use-given-private-key-identity-file/)