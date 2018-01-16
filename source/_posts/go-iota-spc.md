---
date: 2015/07/18 12:34:50
title: Golang 中iota的用法
categories:
- programming
tags:
- golang
- lang
---

Golang中可以使用iota方便的定义复杂的常量结构，下面是golang spec中的说明

Within a constant declaration, the predeclared identifier iota represents successive untyped integer constants. It is reset to 0 whenever the reserved word const appears in the source and increments after each ConstSpec. It can be used to construct a set of related constants:


## 示例1 ##

```
const (  // iota is reset to 0
	c0 = iota  // c0 == 0
	c1 = iota  // c1 == 1
	c2 = iota  // c2 == 2
)

const (
	a = 1 << iota  // a == 1 (iota has been reset)
	b = 1 << iota  // b == 2
	c = 1 << iota  // c == 4
)

const (
	u         = iota * 42  // u == 0     (untyped integer constant)
	v float64 = iota * 42  // v == 42.0  (float64 constant)
	w         = iota * 42  // w == 84    (untyped integer constant)
)
``
const x = iota  // x == 0 (iota has been reset)
const y = iota  // y == 0 (iota has been reset)
```

Within an ExpressionList, the value of each iota is the same because it is only incremented after each ConstSpec:

## 示例2 ##

```
const (
	bit0, mask0 = 1 << iota, 1<<iota - 1  // bit0 == 1, mask0 == 0
	bit1, mask1                           // bit1 == 2, mask1 == 1
	_, _                                  // skips iota == 2
	bit3, mask3                           // bit3 == 8, mask3 == 7
)
```
This last example exploits the implicit repetition of the last non-empty expression list.

iota是一个比较有特色的东西，看完这两个例子基本就知道怎么用了，相当于它是一个从0开始自动增加的变量，可以重复使用，可以用来定义单位的大小，比如KB，MB，GB等
