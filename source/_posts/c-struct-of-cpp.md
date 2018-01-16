---
date: 2015/12/06 12:34:50
title: C语言中的结构
categories:
- programming
tags:
- C
- CPP(C
- Primer
- Plus)
---
![](/images/clang.gif)

结构体一块本科的时候就没有学好，在学习Go的时候学习了Go定义结构体的方式，最近看C Primer Plus的结构体一章，发现Go借鉴了很多C中结构体的东西，并且这本书中讲的结构体和本科书上的结构体部分很不一样，讲的更加容易理解。

这里只记一些最重要的，因为对结构体已经有一定的基础了，虽然很久没有使用过了。

## 结构体的声明和变量定义
声明结构体的关键字是 struct ，结构体的声明是描述结构如何组合的主要方法。下面是一个示例：

```
struct book {
	title char[20];
	char author[20];
	price float;
}

// 定义一个结构体的变量
struct book cpp;
```
我们把struct 后面的 book 称作（可选的）标记，用来引用该结构的快速标记。
<!-- more -->
变量的声明使编译器创建一个变量 cpp ，编译器使用结构体 book 为模板为变量分配空间，这些空间以一个名字 cpp 被组合在一起，它们在一段连续的存储器空间。

在变量的声明中，struct book的作用就和int， float是一样的，可以如下定义一个变量及一个指针：

```
struct book cpp, *pcpp;
```

一种更常见的方式是将结构体的声明和变量定义合并成一步：

```
struct book {
	char title[20];
	char author[20];
	float title;
} cpp, *pcpp;
```
其中的标记 book 是可选的，有book标记会保留到该结构声明的引用，用来声明其他的结构体变量，如果不带book标记，则是一个未命名的结构类型。我以前一直把这里理解错了，以为后面变量定义是类型定义。

## 结构体的初始化
在Go中，结构体可以用类似于js的使用大括号的方法初始化，非常方便，原来C（ANSC）中的结构体也可以这样，因为很久没有使用过C了，所以这些都忘记了（或许以前没教过？？）这让我对C语言更加有好感了，我们可以用很方便的方式初始化一个结构体变量。

```
struct book cpp = {
    "c primer plus",
    "stephen prata",
    60.0
};
```

> 现在（ANSC之后）C的数组初始化也可以使用字面量初始化了，比如 `int a[] = [1,2,3,4]` 这样，非常方便

 
甚至还可以指定要初始化的项目，比如值初始化title和author部分：

```
struct book apue = {
	.title = "APUE",
	.author = "W. Stevens"
}
```
语法有点“独特”，使用点运算符和成员名来标识具体的元素，和Go的直接使用成员名不同。甚至指定初始化和常规的（顺序）初始化可以混合使用，不过常规初始化方法跟在指定初始化方法之后，为其他成员提供初始化，这样规定：对特定成员的最后一次赋值是它实际获得的值，比如：

```
struct book cpp = {
	.price = 61.0,
	.author = "prata",
	60.0
}
```
最后的60.0会赋给price成员而覆盖之前的指定初始化的值。这样的代码虽然基本不会出现，但是最好也知道它的运行原理。


## 结构体数组
我们知道基本数据类型的变量数组直接定义就可以分配空间了，结构体可以看作一种新类型，它也是定义声明变量之后就会自动分配空间的，结构体的数组也是这样。

```
struct book library[10];
```
这样就定义了一个有10个book变量的数组，并且**已经分配了存储空间**。 结构体的数组和普通数组索引方式是一样的。

结构体数组也可以使用字面量初始化方法，如下

```
struct book lib[2] = {
	{"apue", "stevens", 128.0},
	{"cpp", "prata", 60.0}
};
```
是不是很方便了。要注意最外面的是 `{` ，不是 `[` 哦。 对于成员是一个结构的结构体变量，同样可以使用字面量初始化，记住，只是初始化，不能用于对结构体变量的赋值哦。

## 指向结构的指针
指向结构体的指针是一个一直都没有掌握好的点，希望这里能记录好一点，加强理解。

对于指针有几个<del>好处</del>，第一：就像指向数组的指针比数组本身更容易操作一样，指向结构的指针通常也更容易操作； 第二：在早期的C中参数传递只能使用结构的指针；第三：很多奇妙的数据表示都是用了包含指向其他结构的指针的结构。

和数组不同，结构的名字不是该结构的地址（即单独的结构名并不是该结构地址的同义词），必须使用 & 运算符。声明一个指针的方式与一个普通变量没有什么区别：

```
struct book *cpp;
struct book c = {
	"c primer plus",
	"prata",
	60.1
};

cpp = &c;
```
假设 `lib` 是一个 struct book 的数组，现在用结构指针 cpp 指向 `lib[0]`,那么根据指针的运算规则， `cpp+1` 会指向 `lib[1]`。虽然在一般的认识里面，结构体中的元素在存储器中是一次排列的，所以可以根据各个元素的大小来计算 `cpp+1` 与 `cpp` 之间的地址差多少。但是考虑到系统对存储器的对齐要求，不同的系统对齐的方式可能不一样，所以使用各个成员大小相加的方式计算结构的存储大小是不合适的。


## 访问结构的成员
这个比较简单，注意结构和指向机构的指针访问成员的方式不一样，结构本身使用 `.`运算符访问，而指向结构的指针则使用 `->` 访问。

```
strcut book cpp, *pcpp;
...
char *title;
title = cpp.title;
// title = pcpp->title;
// title = (*pcpp).title;  // 因为 . 的优先级比 * 高，必须要有括号
```

## 其他特性
### 赋值结构体
我们可以向普通的 int 类型变量一样对结构体变量使用赋值操作（将一个结构体赋值给另一个结构体）。这是可以的，因为和数组不同，数组的名字是一个指针，所以不能使用直接的赋值来复制一个数组，但是结构体名不是指针，使用赋值可以把另一个结构体初始化成这个结构体样的内容。

```
struct book cpp = {
    "c primer plus",
    "prata",
    60.1
};
struct book c1;
c1 = cpp;
```
c1 和 cpp 现在是两个互不影响的变量。

### 结构还是指针
在代码中，使用结构本身还是使用指向结构的指针是一个比较难以理解的点。使用指针的好处是执行起来很快，但是缺乏对数据的保护。被调用函数可能修改原来结构中的数据，不过 ANSI C中新增了 const 限定词可以解决数据篡改的问题，对于使用了const 修饰的结构体指针参数，如果在函数中修改了原结构的内容，编译器会将此视为一个错误。

```
float buy(const struct book *b, int num) {
	// 不能在这里修改 b 的内容
	return b->price * num;
}
```

### 结构成员使用字符数组而不是指针
在结构的成员中如果有字符串推荐使用字符数组而不是指向字符的指针。对于指向字符串的指针，结构体内只保存一个地址，真正的字符串内容被保存在在编译器存储字符串常量的节中。这样的结构定义很有可能导致一些未初始化变量的问题，

结论就是如果需要一个结构来存储字符串，请使用字符数组成员，存储字符指针有它的用处，但也有被严重误用的可能。

### malloc
在我的印象中，老师教的初始化结构体总是使用malloc函数的，定义一个结构体的指针，再用malloc去分配存储空间，在一个个初始化成员，好麻烦。。。现在更推荐的使用时定义结构体变量，用字面量方法初始化结构体，在用取地址操作获取指针，这样更方便。

在前面讲过的不推荐使用字符指针，如果一定要使用字符指针的话，就需要用malloc先分类一个动态大小的空间给这个指针。这在有的时候还是很有用的，因为可以动态的分配字符串的大小。但有一个问题是，malloc分配的空间都得用free去释放，这样有点麻烦的。

malloc的函数定义如下：

```
void *malloc(size_t size);
```

### 复合文字和结构
在C99中新增的复合文字（复合字面量 compound literal）特性，可以使用复合文字**创建一个**被用来作为函数参数或被赋值给另一个结构的结构。 这种字面量方式使用非常棒，用法和动态语言有点相似，只是需要做一个类似于强制类型转换的操作。

```
struct book csapp;
...

csapp = (struct book) { "csapp",
	"Hallaron",
	99.0
};
```
还可以将这种字面量定义作为函数的参数，或者取它的地址，这都是很方便的用法。

```
buy( &(struct book) {"csapp", "Hallaron", 99.0}, 3);
```
出现在所有函数外面的字面量具有静态存储时期，而出现在一个代码块内部的复合字面量具有自动存储时期。使用于常规初始化项目列表的语法规则同样适用于字面量方式，也就是说可以在字面量中使用**指定初始化项目**。

### 伸缩性数组成员
这是C99提供的一个新特性，一般来说结构的成员都是固定大小的，由于C没有动态长度的数组，所以结构中的数组的大小都是固定长度的，在C99中提出了伸缩数组成员的概念，即在结构定义的最后一个成员可以是一个变长的数组（长度不在结构声明时指定）。这个特性有一点的约束：

－　伸缩型数组成员的必须是最后一个数组成员
－　结构中至少有一个其他成员
－　该成员向普通数组一样声明，不过方括号内为空，不指定程度

声明一个这种结构的变量，不会为伸缩型成员分配任何空间，事实上不应该声明这种结构的变量，而应该使用该给结构的指针，然后用malloc分配足够的空间。这里的空间的计算应该包括动态成员的大小，比如：

```
struct book {
    char title[20];
    float price;
    char author[][20];
};

int main() {
    struct book *csapp;
    csapp = (struct book *)malloc(sizeof(struct book) + 2 * 20 * sizeof(char));
    strcpy(csapp->title ,"csapp");
    csapp->price = 99.00;
    strcpy(csapp->author[0],"Randal E.Bryant");  // 访问伸缩性数组成员
    strcpy(csapp->author[1], "David R. O'Hallaron");
}
```
这里再提一下char数组的初始化，在C中可以在声明char数组的时候使用 `char name[10] = "wuxu";`这样的方式初始化，但是在在定义之外的其他地方不能再使用这中方式初始化，即不能用 `csapp->title = "csapp";` 这样的方式初始化title成员了。这有点麻烦，特别是对于习惯动态语言的人来说。要在定义之外初始化这个成员，要么使用上面的 `strcpy` 函数，要么一个个字符赋值。

伸缩型数组成员有一定的动态性了，但是很明显和动态语言来说，还太弱了，不过C本身就不是动态性的语言，有这样的特性已经是很大的进步了。


## 把结构保存到文件
写文件我们可以使用 `fprintf` 函数实现，但是 fprintf 本身的格式定义太过复杂，对于结构的保存会非常麻烦，我们可以使用 `fread` 和 `fwrite` 函数以结构的大小为单元进行读写。

```
#include <stdio.h>
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
size_t fwite(const void *ptr, size_t size, size_t nmemb, FILE *stream);
```
从两个函数的声明可以看出它们可以读写任意类型的（void *)变量，自然可以用来读写结构体变量，只需要指明结构体的大小和要读写的结构体的个数。这对于我们保存结构体非常方便。不过写入到文件的都是二进制格式，所以用文本编辑器打开大部分都是乱码了。读写结构到文件的简单示例如下：

```
// write
struct book {
    char title[20];
    float price;
};

int main() {
    FILE *f;
    struct book lib[2] = {
        {"csapp",
            99.0
        }, {"cpp",
            60.0
        }
    };
    if ((f = fopen("books.dat", "a+b")) != NULL) {
         rewind(f);
         int wsize = fwrite(lib, sizeof(struct book), 2, f);
         printf("write to file of size: %d\n", wsize);
    }
}

// read
struct book {
    char title[20];
    float price;
};

int main() {
    FILE *f;
    struct book books[6];
    if ((f = fopen("books.dat", "r+b")) != NULL) {
        rewind(f);
        fread(books, sizeof(struct book), 2, f);
        for (int i=0; i<6; i++)
            printf("%d: %s\n", i, books[i].title);
    }
}
```

结构体先到这，下面的联合，枚举的类容新写一篇。

参考：

- [http://stackoverflow.com/questions/11515287/c-variable-has-initializer-but-incomplete-type](http://stackoverflow.com/questions/11515287/c-variable-has-initializer-but-incomplete-type)
- The C Library Reference Guide
