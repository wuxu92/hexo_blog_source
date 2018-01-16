---
date: 2015/11/15 12:34:50
title: C链接不同的 .o 编译可执行文件
categories:
- programming
tags:
- c
---

一个方法可以有很多种实现，在a.c种实现func函数，在b.c种也实现func函数，在main中使用了func，可以通过链接时链接到不同的.o文件调用不同的实现。可能描述的不是很清楚，下面用一个例子说明。

现在要实现一个三个整数轮转的函数，就是 int x, y, x要调换他们的值，也就是向右轮转，是y=x, z=y, x=z; 下面有两种实现

a.c

```
void decode(int *xp, int *yp, int *zp) {
    int tp1 = *xp;
    *xp = *zp;
    int tp2 = *yp;
    *yp = tp1;
    *zp = tp2; 
}
```

b.c

```
void decode(int *xp, int *yp, int *zp) {                                                       
    int tx = *xp;
    int ty = *yp;
    int tz = *zp;

    *xp = tz;
    *yp = tx;
    *zp = ty;
}
```
他们实现了同样的功能，现在在main种调用decode方法。

main.c

```
#include <stdio.h>

int main() {                                                                                   
    int x = 1;
    int y = 2;
    int z = 3;

    decode(&x, &y, &z);
    printf("%d, %d, %d\n", x, y, z);
}
```
下面就是编译了，使用gcc分别链接到a的.o文件和b的.o文件。
先汇编到 .o 文件。

```
gcc -c a.c
gcc -c b.c
```
这样会生成 `a.o` 和 `b.o` 两个文件，下面编译main.c 生成可执行文件。

```
gcc -o decode1 a.o main.c
gcc -o decode2 b.o main.c
```
这样生成两个可执行文件`decode1` 和 `decode2`，分别运行他们可以得到其结果是一样的，都输出 `3, 1, 2`

很久没有用C了，很多东西都忘记了。
