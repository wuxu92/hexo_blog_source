---
date: 2015/12/07 12:34:50
title: C 联合、枚举及函数指针
categories:
- programming
tags:
- C
- CPP(C
- Primer
- Plus)
---
![](/images/clang.gif)
## 联合（union）
联合是一个能在同一存储空间里（但是**不是同时**）存储不同类型数据的数据类型。用来存储某种即没有规律，事先未知顺序的混合数据类型。联合使用于结构同样的方式建立，也需要一个联合模板和联合变量，下面是一个例子：

```
union hold {
	int digit;
	double bigfl;
	char letter;
};

union hold fit;
```
一个hold结构可以含有一个int型数值**或者**一个double型数值**或者**一个char型数值。一个联合的变量可以存储模板中声明的成员**的某一个**。编译器分配空间时，会分析各成员需要的空间，然后按最大需求分配。因为联合**一个时刻只存储一个值**，所以初始化与结构并不一样。

```
fit.letter = 'A';
fit.digit = 127;
union hold holdA = { .bigfl = 10.08 };
```
要注意的是，**需要程序员维护存储在联合变量的是哪一个成员**，有时候使用了一个成员保存值，而使用另一个成员来查看这些内容，这有时很有用，但需要是程序员管理下进行的，否则会导致错误。

## 枚举（enumerated type）
枚举类型声明代表整数常量的符号名称，通过使用关键字 enum 可以创建一个新的类型，并可以指定它**可以**具有的值，实际上enum常量是int类型的，因此在任何使用int类型的地方都可以使用它。枚举使得程序有更高的可读性，它的语法与结构很相似。

```
enum spectrum { red, orange, yellow, blue };
enum spectrum color;

// 可以当作int使用
for (color = red; color <= blue; color++)
	...

```
类型声明中的 red, orange 等我们称之为 **枚举常量**，规定枚举常量都是int类型的，而定义的变量 color 称为枚举变量，可以较为宽松地限定为任意一种整数类型，常有编译器决定，比如我们这里color的范围在0-3之间，可以选择为unsigned char表示。要注意对枚举变量的一元操作符在C++中是不允许的，如果代码可能迁移到C++则不应该这样用。

枚举常量默认从0开始递增，也可以为枚举列表中的常量指定特定的值。如：

```
enum level { low = 100, medium = 101, high };
```
这样后面没有指定值的常量会赋予后续的值（即high为102）。Go中有更加强大的 [itoa](http://wuxu92.github.io/go-iota-spc/) 生成更加复杂的赋值模式。

> 可以字啊同一个作用域内对一个变量和一个标记使用同一个名字而不会发生错误，但是不能在同一作用域内使用名字相同的两个标记或者变量。

## typedef
typedef 是一种高级数据特性，它能够为某一类型创建自己的名字，与#define有3个不同之处

- typedef 给出的符号名称仅限于对类型，而不是对值
- typedef 的解释有编译器执行，而不是预处理器（也就是说 #define 由预处理器执行
- 虽然typedesf的范围有限，但在受限范围内，typedef比 #define更灵活

下面是一个使用示例：

```
typedef unsigned int BYTE;
BYTE x, y[2], *z
```
typedef 的作用域取决于typedef语句所在的位置，它的作用域与变量相似。通常typedef定义的类型使用大写字母，以提醒用户这个类型名称实际上以一个符号缩写（当然也可以使用小写字母）。


实际上，在 <sys/types.h> 中定义的很多很多类型，比如 time_t, size_t 都是使用typedef定义的整数类型。这是由于C标准留给具体实现来决定使用哪种类型。

注意typedef和#define的区别：

```
typedef char * STRING
STRING name, sign; // char *name, *sign;

// define
#define STRING char *
STRING name, sign; // char *name, sign;
```
一种更常用的 typedef 场景是对strut 结构体创建新的类型：

```
typedef struct book {
	char title[20];
	float price;
} Book;
Book cspp;
```
这样使用更见方便，也更见常见。

## 一些奇特的声明
这里记录几个比较有用的：

```
int *reisks[10];		// 表示具有10个元素的数组，每个元素是一个指向int的指针
int (*risks)[10];		// 一个指针，指向具有10个元素的int数组
int *oof[3][4];			// 一个 3×4的数组，每个元素是一个指向int的指针
int (* uuf)[3][4];		// 一个指针，指向 3×4的int数组
int (* uof[3])[4];		// 一个具有3个元素的数组，每个元素是一个指向具有4个元素的int数组的指针
```
理解这部分，需要知道， [] 和 () 具有相同的优先级，其优先级比 * 高，且 [] 和 () 都是从左到右进行结合的。这一块确实是比较难以理解的，过一段时间又会混乱掉。

再看一下复杂的函数返回值类型：

```
char * fump();			// 返回指向char的指针**的函数**
char (* frump)();		// 指向返回类型为char的**函数的指针**
char (* flump[3])();	// 由3个指针组成**的数组**，每个指针指向返回类型为char**的函数**
```
一个技巧是，首先确定类型，是数组、指针还是函数，在确认其他的。


更复杂的typedef应用：

```
typedef int arr5[5];		// arr5 是一个有5个元素的int数组类型
typedef arr5 * p_arr5; 		// p_arr5是指向具有5个元素的int数组的指针 的类型
typedef p_arr5 arrp10[10];	// 一个具有10个元素的数组类型，每个元素是一个指向有5个元素的int数组
```
## 函数和指针
上面一节中，函数指针是比较难懂的，之前也确实没有用过C中的函数指针，在Go，JS甚至PHP中都用过相似的概念，这里学习一下C中的函数指针。

对于一个函数的指针，必须声明它指向的函数的类型，在C中函数的类型包括返回类型及函数的参数类型。考虑下面的函数声明：

```
void ToUpper(char *);
```
描述这个函数声明就是：一个具有char * 类型的参量，返回类型是void的函数，那么要声明这种函数的指针，可以如下：

```
void (*pf)(char *)
```
pf 就是一个指向 ToUpper类型函数的指针了。其前一个括号是结合符，后一个括号是表示这是一个函数指针，并且用于说明其参数类型。这是可以用 `*pf` 代替函数名 ToUpper。 第一个括号是不可省略的，之前的文章里已经提到过，`void *pf(char *) `表示一个返回一个指针的函数。

有了函数指针后，可以把适当类型的函数的标识（地址）赋值给它，函数名即是函数的地址：

```
char mis[12] = "how are you";
pf = ToUpper;  // 不需要 ToUpper()
(*pf)(mis);

pf(mis);
```
像上面的两种使用方式在ANSI C中都可以，第一种是C和Unix开发者使用的，第二种是Berkeley的Unix扩展这采用的，而K &R C不允许使用第二种形式，ANSI C保持了对两种方式的兼容。一个使用函数指针的示例：

```
void ToUpper(char *str) {
    while (*str != 0) {
        if (*str >= 'a' && *str <= 'z')
            *str -= 32;
        str++;
    }
}

void ToLower(char *str) {
    while (*str != 0) {
        if (*str >= 'A' && *str <= 'Z')
            *str += 32;
        str++;
    }
}

void show(void (*fp)(char *),char *str) {
    fp(str);
    printf("str after fp: %s\n", str);
}

int main() {
     char *hi = "how are you?";
     void (*fp)(char *);
     fp = ToUpper;	// 类型相同的函数才能赋值给该指针
     show(fp, hi);	// 函数指针作为参数
     fp = ToLower;
     show(fp, hi);
     return 0;
}
```
可以使用typedef简化函数指针的声明，比如使用：

``` 
typedef void(*V_FP_CHARP) (char *);
void show(V_FP_CHARP, char *);

V_FP_CHARP pfun;
```
这样的typedef用法和之前简单的 `typedef unsigned char BYTE` 形式不太一样，不过结合前面对函数指针的介绍，应该也可以理解了。

完。

参考：

- 除了 C Primer Plus，然而并没有
