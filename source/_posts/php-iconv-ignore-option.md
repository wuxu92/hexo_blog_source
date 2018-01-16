---
title: php字符集转转换失败的问题
date: 2016-03-30 17:58:03
categories:
- programming
tags:
- php
- programming
---

今天科委的项目有子系统报错说有几条数据上传不了，各种排查终于搞好了。

首先是ESB的服务器他们重启了，然后IP没配对，端口没配对，我们curl一直是失败的，导致我们的nginx返回 504 gateway timeout。这个问题之前[有一篇文章](/php-fpm-504-error/)已经说过了。php执行curl的时间超过了nginx配置的响应时间，就会报 upstream time out 然后给客户端返回 504 。

等ESB服务起好了，深圳那边有一条数据始终传不上了，诡异的是ESB根本没有收到数据。我把我们转发的PHP的日志记录改了好多次，终于发现是在某一步上传的内容被置空了，所以在我们转发的时候，其实并没有把子系统的数据发到 ESB。

调试了一下，发现是 iconv 执行失败了。 不同的系统对接麻烦的就是字符集的问题，我们的项目一直都是用 UTF-8的，可是很多的java项目都使用GBK，东华的ESB系统也是使用GBK编码的，所以我们收到数据后要执行一次字符集转换才能转发到ESB：

```php
$xml = iconv('UTF-8', 'GBK', $xml);
```
就是这一步，iconv转换失败的时候会返回false而不是报错，也没有日志记录，所以 $xml 被置为 false 了。。。。。至于为什么会失败，我也不知道，应该是有某种特殊编码把，有一个办法可以跳过这个转换失败,就是在目标字符集加一个选项：

```php
$xml = iconv('UTF-8', 'GBK//IGNORE', $xml);
```
领略到我大 PHP 神奇的方法调用了么~~~


参考：

- [http://php.net/manual/en/function.iconv.php](http://php.net/manual/en/function.iconv.php)