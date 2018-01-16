---
date: 2015/09/04 12:34:50
title: 更新gitlab到7.14.1
categories:
- server
tags:
- git
- gitlab
- linux
---

去年为实验室部署了gitlab用于项目代码和文档的管理，在hixicar项目中用的比较多，也起到不错的作用，可惜其他项目就没怎么用，基本是我自己在用了，不过也极大方便我自己的项目的管理。。
当时安装的最新版本是7.3,作为一个版本帝软件，gitlab的小版本号几乎一直在涨，最近已经涨到了7.14.1了。用的7.3虽然可用，但是有一些小小的问题，当一个项目的文件数特别多，hixi的文件大概到了1G多，后来把文档分离出来后还有500M左右，这样这个项目的Graph画不出来了。虽然没什么太大影响，但是总觉得不爽，一直也没有去解决，最近没什么事就想着升级一下。

官方提供的升级非常方便，可以直接使用软件仓库安装gitlab-ce替换原来安装的gitlab，会保留原来的数据不丢失。简直碉！堡！了！哇！

官方的升级文档： [https://about.gitlab.com/upgrade-to-package-repository/](https://about.gitlab.com/upgrade-to-package-repository/)

## 安装repo ##

```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```
但是我的curl要使用代理，之前使用代理是配置`-x`参数，但是从网上下载的这个 script.rpm.sh 脚本里面也要用curl去下载文件，这样就会导致运行这个脚本失败。可以设置curl的全局代理，这里有一个小小的trick：使用alias；

```
alias curl='curl -x proxy_host:port'
```
这样使用curl就相当于使用curl的别名（alias）会带上代理设置了。另一个方法就是在$HOME/.curlrc 配置文件中添加代理：

```
vim ~/.curlrc
// 添加
proxy=proxy_host:port
```
这样就可以了。
其实使用命令安装repo挺麻烦的，尤其在网络不方便的情况下。可以直接使用下面的文本添加repo到/etc/yum.repo.d/

```
[gitlab_gitlab-ce]
name=gitlab_gitlab-ce
baseurl=http://packages.gitlab.com/gitlab/gitlab-ce/el/6/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt

[gitlab_gitlab-ce-source]
name=gitlab_gitlab-ce-source
baseurl=http://packages.gitlab.com/gitlab/gitlab-ce/el/6/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
```
上面是对应centos 6的，7的话自己相应修改就可以了。其中的http协议也可以改成https。

## 安装gitlab-ce ##
gitlab提供的开源免费版本命名是gitlab-ce了，包名也成了gitlab-ce，安装好repo后，网络比较好的话，可以直接运行安装:

```
sudo yum install gitlab-ce
```
仔细看输出会看到gitlan-ce会替换原来的gitlab。因为gitlab的安装路径固定在/opt/gitlab/, 所以替换安装也比较方便吧。反正觉得真的挺方便的，原来的repositories都还在。

但是，gitlab-ce这个包有200多MB，使用yum安装下载数据要300多M。我试着下了好久都没有下载下来，只好在本地windows用浏览器下载了（本机翻墙速度比较好，500-1000Kb/s），再scp到服务器。

rpm下载地址： [https://packages.gitlab.com/gitlab/gitlab-ce?filter=rpms](https://packages.gitlab.com/gitlab/gitlab-ce?filter=rpms)

scp到服务器,可以使用git客户端提供就有的scp工具，启动git bash，运行 `scp gitlab-ce-7.14.1-ce.0.el6.x86_64.rpm username@host:gitlab-ce-7.14.1-ce.0.el6.x86_64.rpm`

在服务器端使用rpm安装

```bash
cd
sudo yum localinstall gitlab-ce-7.14.1-ce.0.el6.x86_64.rpm
```

使用rpm包本地安装也会替换安装原来的gitlab，简直不错。
要注意，虽然git仓库相关的数据不会被删除，但如果你修改多gitlab的rails项目，比如我修改过登陆页面的内容，这些修改会被覆盖掉。如果要保留这些数据要记得做备份。而gitlab的配置文件， /etc/gitlab/gitlab.rb 应该是不会被覆盖的。

安装完后，gitlab会自动 reconfigure 并启动，如果有问题就需要自己看输出内容排查了。比如我们使用学校的smtp服务器发送邮件，修改了gitlab配置的smtp配置，而又在/etc/gitlab/gitlab.rb 中配置了邮件相关的项，导致新的系统 reconfigure 失败，把两处的配置修改到相同就可以正常启动了。[参考](http://stackoverflow.com/questions/26684035/gitlab-smtp-config-exception)

新的gitlab添加了很多新的特性，更多的配置项，甚至还有多套主题。。。还不错。

加张图

![](http://wuxu92.github.io/images/blog-article-images/blog/gitlab7.14.png)

