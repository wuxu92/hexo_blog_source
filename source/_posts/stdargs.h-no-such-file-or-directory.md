---
date: 2015/12/10 12:34:50
title: fatal error： stdarg.h： No such file or directory
categories:
- programming
tags:
- C
- Liux
- gcc
---

今天学习C的库函数，到可变参数部分，使用gcc编译一个简单的程序发现编译报错，代码如下：

```
#include <stdargs.h>
#include <stdio.h>

double sum(int lim, ...);
int main(void) {
    double s, t;
    s = sum(3, 1.0, 2.02, 3.123);
    t = sum(8, 2.9, 9.1, 2.2, 4.1, 99.2, 43.2, 54.1, 9.0);

    printf("sum of s is %g\nsum of t is %g\n", s, t);
}
double sum(int lim, ...) {
    va_list ap;
    double sum = 0;
    int i;
    va_start(ap, lim);
    for (i=0; i<lim; i++)
        sum += va_arg(ap, double);
    va_end(ap);
    return sum;
}
```

错误如下：`fatal error: stdarg.h: No such file or directory`，头文件不存在，很奇怪，去找了一下 /usr/include 目录确实没有这个文件，<del>用find查找竟然也没发现这个文件，难道系统没有这个文件？不可能的啊。</del>(是find 的时候打错名字了。。。真是见鬼，闹腾到今天才发现）

实际上系统是有这个文件的，不过不在 include 目录下，在 `/usr/lib/gcc/x86_64-redhat-Liux/4.8.2/include/` 这个目录下是有这个文件的。这是我的CentOS下的路径，可能其他发行版的路径名略有不同。

为了让gcc找到这个头文件，我们可以拷贝一份到 `/usr/include` 下，但是更好的方法是做一个软连接：

```
sudo ln -s /usr/lib/gcc/x86_64-redhat-Liux/4.8.2/include/stdargs.h /usr/include/stdargs.h
```
在编译就没问题了。

软连接在这种情况下很有用的工具，很多编译器找不到共享目标啊，找不到头文件啊，都给个软连接可以解决。

== added

前面讲到的find 查找文件，因为centos 7没有了locate命令（locate真的很方便，只是要维护一个db确实比较不讨人喜欢就被砍了吧），查找文件都是用find，常用的方法如下：

```
find -L /usr -name "*file-to-find*" -type f
```
-L 表示 follow symbol link， -type f表示输出文件，不输出目录，也可以 -type d 只输出目录。

查找的名字，可以用双引号括起来，加通配符。
