---
title: 从Jekyll迁移到Hexo
date: 2016-01-07 14:28:17
categories:
- self
tags:
- jekyll
- hexo
- fe
- blog
---
[前一篇](/jekyll-posts-to-hexo-style/) 讲了为什么要从Jekyll迁移到Hexo，以及将Jekyll文章转换到Hexo风格的实现，这里记录一下迁移的具体过程。

[hexo](https://hexo.io/zh-cn/)是使用nodejs开发的静态博客生成框架，我经常在Windows下写博客，然后push到github，然后在github.io查看，这里主要介绍Windows下的迁移。

## 安装node.js 和 hexo
首先要安装Node.js，现在各个平台都支持了，Windows下的安装也很方便，直接去 [https://nodejs.org/en/download/stable/](https://nodejs.org/en/download/stable/) 下载最近的版本，Windows的是一个 msi 文件，就像安装一个软件一样，点点下一步就可以了。
<!-- more -->

安装程序会自动添加环境变量，如果没有的话，自己添加一下，麻烦的是Windows的环境变量修改需要重启才能生效，真是太不给力了。

安装后就可以在命令行使用Node.js了，我们使用git bash作为命令行工具，如果你还在使用cmd的话，推荐你去试试 git bash，可以或者Linux bash 相近的使用体验，比cmd强太多了,并且只要安装官方的客户端就可以了。

安装hexo也很简单：

```bash
npm install hexo-cli -g
cd /path/to/store/blog/files
hexo init blog
cd blog
npm install
hexo s
```
第一行就是安装hexo程序，npm的安装比ruby的gem(jekyll使用的)要方便，gem在国内要修改源才能获得比较好的速度，npm貌似速度还不错，如果要设置代理可以用下面的命令：

```bash
npm install hexo-cli -g --peoxy=http://proxy-server:port --verbose
```
最后的 --verbose 会输出执行的信息，如果安装失败或者很久都没有反应可以加上这个参数看在哪里卡住了。

第5条命令，`npm install` 是npm的安装命令，会自动在blog目录下寻找 package.json 作为配置文件安装必要的模块，安装完后就可以启动hexo服务器了。这里的安装也可以像上面那样指定代理服务器。

## 安装主题
Hexo有很多不错哦主题，我用的是比较流行的 NexT 主题,它的官网是： [http://theme-next.iissnan.com/](http://theme-next.iissnan.com/)。使用如下：

```bash
cd /path/to/hexo/site
git clone https://github.com/iissnan/hexo-theme-next themes/next
```
然后修改站点配置文件，将 `theme` 字段更改为 `next`。该主题的更多内容参考: [http://theme-next.iissnan.com/five-minutes-setup.html](http://theme-next.iissnan.com/five-minutes-setup.html)

## 额外的页面
一般来说，博客除了文章，标签页面外还需要一些 关于啊，个人简历类的页面，Hexo使用下面的命令生成：

```
hexo new page "about"
```
然后编辑生成的 source/about/index.md 文件，可以直接写html内容，一般把这些页面的评论关掉，在 front-matter 部分添加 `comments: false` 即可。

## 迁移文章
jekyll文章的迁移可以先参考[官方的文档](https://hexo.io/zh-cn/docs/migration.html)，不过如果原来的文章比较多，内容比较诡异的话，可以用 [前一篇](/jekyll-posts-to-hexo-style/) 的方法，或者自己再修改一下后批处理。

将文章迁移后，试试能不能渲染成功：

```bash
hexo clean
hexo g
```
g 就是 generate 的缩写，还有 s 是server的缩写， d 是deploy的缩写，很方便。如果生成出错了就需要自己找找原因了，js的报错还是很麻烦，很难找到问题的原因，一般是文章的代码段，双大括号出错了，主要是没有办法知道具体是哪个文件的问题，所以文章很多的时候很难找到。

## 部署到github
hexo提供自动部署的功能，这简直比Jekyll方便太多了，不过Hexo和Jekyll不同的是，Jekyll是直接把Jekyll的工程push到github后由github去编译生成最终的静态内容，而Hexo的部署是本地生成好静态文件push到github。这样的好处是push上去后立即能看到效果。

部署是配置文件的deploy部分，先参考官方文档： [https://hexo.io/zh-cn/docs/deployment.html](https://hexo.io/zh-cn/docs/deployment.html)，一般的配置如下：

```bash
deploy:
  type: git
  repo: git@github.com:yourname/yourname.github.io.git
  branch: master
```
配置好后，部署的命令是：

```bash
hexo g -d
// 或者直接
hexo d
```
g表示generate, -d表示 deploy，`hexo d -g` 也是一样的。

这里假设你已经配置好了github的密钥，就能自动部署不需要输入密码了，如果需要输入密码说明你的github账号没有本机的公钥。添加一下就好了。

Hexo能玩的东西还挺多的，next主题也提供了很多配置的，比如多说，disqus评论，百度和Google的统计等，只需要配置一些就可以了，非常方便。

## CNAME
如果你的github pages配置了CNAME 的话，需要将CNAME文件放到 hexo_size/source 下，否则每次部署的时候会丢失，导致不能通过你自己的域名访问了，CNAME是一个只有一行的文本文件，Windows下可以用 git bash 的 vim 生成这种没有后缀的文件。它的内容就是你自己配置的域名，比如我的就一行：

```
blog.wuxu92.com
```

这样部署后就能通过自己的域名访问了。

好了，迁移完成了，好好写文章吧。

参考：

- [https://hexo.io/](https://hexo.io/)
- [http://theme-next.iissnan.com/](http://theme-next.iissnan.com/)
- [http://hadb.me/tags/Hexo/](http://hadb.me/tags/Hexo/)
- [https://hexo.io/zh-cn/docs/troubleshooting.html](https://hexo.io/zh-cn/docs/troubleshooting.html)
- [http://zhiho.github.io/2015/09/29/hexo-next/](http://zhiho.github.io/2015/09/29/hexo-next/)
- [https://github.com/iissnan/hexo-theme-next/wiki/%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE%E5%8F%82%E8%80%83](https://github.com/iissnan/hexo-theme-next/wiki/%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE%E5%8F%82%E8%80%83)