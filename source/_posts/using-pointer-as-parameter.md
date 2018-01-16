---
date: 2015/12/08 12:34:50
title: C中使用指针作为参数的一些问题
categories:
- programming
tags:
- C
---
这个问题其实比较常见了，但是经常会出错，就是C中函数的参数是值传递的，即使对于指针，也是值传递。很多高级语言在这一点是相通的。

今天看到这个问题是在 c-fqa.com 上的一个问题，看下面的代码：

```
void f(int *ip) {
	static int dummy = 5;	// static make dummy accessiable for whole file
	ip = &dummy;
}

int *ip;
f(ip);
```
这时候ip指向哪里呢？实际上ip可能指向任何地址。因为ip没有初始化，传递给f函数调用的参数只是指针值的**拷贝**，对拷贝的修改并不会改变 ip 本身的内容。要达到本来的目的，需要传入 ip的地址，也就是指针的指针作为参数：

```
void f(int **ipp) {
	static int dummy = 5;	// if no static here, then ip can be anywhere, *ip can be any value
	*ip = &dummy;
}

int  *ip;
f(&ip);
```
另一种方法是把结果的指针作为函数的返回值，这是更常用的方式：

```
int *f() {
	static int dummy = 5;
	return &dummy;
}
int *ip = f();
```
另一个例子是，定制一个 malloc 的自己的版本：

```
void mymalloc(void *retp, size_t size) {
	retp = malloc(size);
	if (retp == NULL) {
		fprintf(stderr, "out of memory\n");
		exit(EXIT_FAILURE);
	}
}
```
看起来没有什么问题，但是在调用这个函数时并不能正常工作，因为被调用函数接受的指针是调用者传入的指针的拷贝。

> C always uses pass by value. You can simulate pass by reference yourself, by defining functions which accept pointers and then using the & operator when calling, and the compiler will essentially simulate it for you when you pass an array to a function 


这些内容在Java和Go中也是通用的，指针作为参数实际上也是值传递的哦。


参考：

- [http://c-faq.com/ptrs/passptrinit.html](http://c-faq.com/ptrs/passbyref.html)
- [http://c-faq.com/ptrs/passbyref.html](http://c-faq.com/ptrs/passbyref.html)
