---
date: 2015/12/08 12:34:50
title: malloc的强制类型转换
categories:
- programming
tags:
- C
---

在之前的认识里面，`malloc` 函数分配一定大小的存储空间，返回一个 `void *` 类型的指针，一直都是要对返回类型做显示的类型转换，但是今天在 stackoverflow 上面看到一个提问，提到这个强制的类型转换是**不建议使用的**。这是为什么呢？

一般我们这样使用 malloc :

```
int *sieve = (int *)malloc(sizeof(int) * length);
```

我们知道malloc返回的是一个 `void *`，即指向任何类型的指针，在这种情况下，返回的指针能自动地安全地提升到任何其他类型的指针类型，这可以是一个隐式的类型转换。

加上显式的类型转换后，代码显得更加混乱，尤其是当指针类型很复杂的时候，显示的类型转换就不是那么好理解，并且容易出错。

在没有 `#inlcude <stdlib.h>` 就使用malloc时，编译器会认为malloc的返回值是 int 类型，如果不使用显式的类型转换，编译器会抛出一个隐式类型转换警告，而如果使用了显式类型转换则不会抛出这个警告（而导致潜在的错误）。


> Suppose that you call malloc but forget to #include <stdlib.h>. The compiler is likely to assume that malloc is a function returning int, which is of course incorrect, and will lead to trouble. 


事实上，很多人认为，虽然显式的类型转换也可以正常工作，但是这并不代表这么做是正确的。包含显式的转换是一种**错误**的做法，因为会存在一些潜在的风险，上面的代码是一种很不好的风格，首先是不需要的显示类型转换，其次是重复了不需要的类型信息 `int`，更好的做法是直接使用变量的间接引用（dereference）作为类型信息，推荐的使用方式如下：

```
int *sieve = malloc(length * sizeof *sieve);
```

> sizeof 后面的括号只有当操作数是类型时才是必须的，后面是变量时，大括号不是必须的，因为sizeof是一个操作符，而不是一个函数

然而，只是C不推荐使用强制的类型转换，而C++仍然需要显示转换，这真是忧伤。不过在C++中应该使用 `new` 而不是malloc，并且不应该使用C++编译器来编译C程序。并且C/C++本来就不是完全兼容的，并不推荐用一份代码去满足两份标准。

其实说这么多，malloc的显式类型转换到底写不写还是个人习惯，只是应该知道，**可以不写，并且应该不写**，推荐不要使用显式类型转换了。

GUN C：

> You can store the result of malloc into any pointer variable without a cast, because ISO C automatically converts the type void * to another type of pointer when necessary. But the cast is necessary **in contexts other than assignment operators** or if you might want your code to run in traditional C.

ISO C11：

> The pointer returned if the allocation succeeds is suitably aligned so that it may be assigned to a pointer to any type of object with a fundamental alignment requirement and then used to access such an object or an array of such objects in the space allocated (until the space is explicitly deallocated)


参考：

- [http://stackoverflow.com/questions/605845/do-i-cast-the-result-of-malloc](http://stackoverflow.com/questions/605845/do-i-cast-the-result-of-malloc)
- [http://stackoverflow.com/questions/571945/getting-a-stack-overflow-exception-when-declaring-a-large-array](http://stackoverflow.com/questions/571945/getting-a-stack-overflow-exception-when-declaring-a-large-array)
- [http://www.gnu.org/software/libc/manual/html_node/Basic-Allocation.html](http://www.gnu.org/software/libc/manual/html_node/Basic-Allocation.html)
- [http://c-faq.com/malloc/mallocnocast.html](http://c-faq.com/malloc/mallocnocast.html)
