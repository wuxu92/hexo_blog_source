---
date: 2015/12/24 12:34:50
title: C函数指针使用
categories:
- programming
tags:
- c
- c-faq
- pointer
---
C 的函数指针前面已经简单介绍过，常用的函数指针及其变量声明方式如下：

```
typedef int func(int, int);

int (*fp)(int, int);
fp = func;

func *fpt；
fpt = func;

int func(int a, int b) {
	return a+b;
}
```
上面的 fp 和 fpt 都是一个指向函数func的指针了，这就有一个问题，它们都是指向函数的指针，那么 *fp 就是本身，在它后面加上参数列表就是一次函数调用了。其形式如下：

```
int a = 1;
int b = 2;
a = (*fp)(a, b);
```
这种调用看起来太麻烦了，实际上函数经常可以**通过指针调用**，实际的函数名也总是隐式地转换为指针（因为这种隐式的转换，所以函数指针的取地址与间接引用经常被忽略）。所以可以直接通过指针调用：

```
a = fp(a, b);
```
这样看起来清晰多了。ANSI C 使用后一种解释，即显示地间接引用（*）不是必需的，虽然仍然允许使用这种方法

下面是一个简单示例：

```
typedef int Pfunc(int);

int main(void) {
    int r, (*fp)(int), func(int);
    Pfunc *fpt;
    fpt = func;

    r = 1;
    fp = func;
    r = fp(r);
    printf("r after fp is %d\n", r);
    r = (*fp)(r);
    printf("r after (*fp) call is %d\n", r);
    r = fpt(r);
    printf("r after Pfunc is %d\n", r);
    return 0;
}

int func(int a) {
    return a * 2;
}
```

参考：

- [http://c-faq.com/ptrs/funccall.html](http://c-faq.com/ptrs/funccall.html)
- [http://stackoverflow.com/questions/840501/how-do-function-pointers-in-c-work](http://stackoverflow.com/questions/840501/how-do-function-pointers-in-c-work)
- [http://c.learncodethehardway.org/book/ex18.html](http://c.learncodethehardway.org/book/ex18.html)
