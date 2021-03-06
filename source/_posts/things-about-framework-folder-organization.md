---
title: Web框架（PHP）文件组织的一些问题
date: 2016-04-27 21:30:20
categories:
- programming
tags:
- php
- framework
---

PHP有很多的web的框架，熟悉基本的PHP开发后再看一两个框架的核心，一般都能搞一个PHP框架出来。一些框架提供的源文件组织方式是一个很好的学习示例，但是很多小团队开发的框架却经常组织混乱，隐藏了很大的问题。

最近Carry（音）又在做一个项目，让我过去帮帮忙，用的是他自己的“框架”，看起来他对于自己的作品还很有信心和自豪感，但是这两天接触了一下，发现问题很大，这里记录分析一下。

这个框架使用了命名空间，这是很好的，在之前的文章中也推荐了这种做法，希望所有的PHP开发者都能使用命名空间，不管了PSR-0还是PSR-4的。而这个虽然使用了命名空间，命名空间的组织却混乱不堪，好像既不符合PSR-0又不符合PSR-4。。。。。这就尴尬了。

当然，我对于这个框架并不是完全熟悉，可能一些有点还没有发现，但是对于文件夹的组织却问题极大，它的文件夹组成大概是这个样子的：


```
|-assets
  |-js
  |-css
  |-img
|-boots
|-lib
|-config
|-entity
|-ksc
  |-module-1
    |-part1.1
      |-***.php
  |-module-2
    |-part2.1
      |-***.php
|-tmp
  |-cache
  |-...
|-tpl
  |-module-1
    |-part1.1
      |-***.tpl
  |-module-2
    |-part2.1
      |-***.tpl
|-module-1
|-module-2
```
<!-- more -->
一看就很复杂吧。。。其中assets是一些静态文件，boots里面是注册自动加载方法的，lib是包含框架核心代码，config是一个配置类，包括数据库等等，entity是实体，对应到数据库表；ksc组织核心业务代码，按照模块分不同文件夹；tmp是临时文件夹，包括cache, compile, session, log等内容；tpl则是smarty使用的模版文件加，也是按照模块组织文件夹；剩下的是各个模块有自己的文件夹。

首先一个很严重的问题就是部署。假设项目路径为：/var/share/nginx/html/project/src/，按照现在的文件夹组织，需要将部署的host的root指定为上面的路径，这样才能请求到各个模块，但是这样也意味着能狗请求各个模块之外的文件！比如tpl、config、lib 甚至 ksc 下的文件！只要知道目录组织就能够直接请求执行这些php文件，而tpl里面的文件请求会直接被当作文件下载！

当然我们可以通过nginx的配置，只处理对应到各个模块的请求，但是这样的配置会非常麻烦，当添加新的模块的时候需要更新nginx配置，这对于维护人员来说简直是噩梦！并且现在这样的组织文件夹过于多了，记忆负担比较大，而命名空间也只能说是聊胜于无。

一种常见的做法是会在框架中独立一个文件夹，比如 `www`，或者 `html` 专门用来处理nginx发来的请求，通常里面只有一个 `index.php`和静态文件，通过`index.php`文件将请求转发到不同的模块、控制器、方法，部署的时候将host的root指定到该 `www` 目录，这样nginx就不能直接请求到框架和业务代码所在的文件了。
  
看了这个框架又想起之前在掌游时自己捣鼓的框架 **ZP**，有时间再搞出来玩玩吧，感觉用来写api还是不错的。
