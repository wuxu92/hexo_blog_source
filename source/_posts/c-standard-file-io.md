---
date: 2015/12/11 12:34:50
title: C 文件IO
categories:
- programming
tags:
- C
- IO
---
![](/images/clang.gif)
## 文件
一个文件可以理解为磁盘上的一段命名的存储区。对于操作系统来说，文件更加复杂，比如大文件可能存储在分散的区段中，或者还会包含一些使操作系统确定文件类型的附加数据等。但是对于C程序员来说不需要考虑底层文件系统的实现，C将文件看出是连续的字节序列，每一个字节都可以单独地存取。 ANSI C提供了文件的两种试图：文本视图和二进制视图。

文本试图和二进制试图是指使用C打开文件的模式，与Windows中的文本文件和二进制文件是两个不同的概念（对C程序来说文本文件和二进制文件没有区别）。使用文本模式打开文件时，会将本地环境表示法映射为C试图，比如 windows 和 Linux有不同的换行模式，但是是用文本模式打开文件时，这两个换行方式都会被映射到 `\n` ，同样一个C中以 `\n` 换行的字符串写到文本模式打开的文件时，会自动转换到 `\r\n` 或者 macintosh 的 `\r`。而使用二进制模式打开文件则不会有这样的映射。通常使用文本模式处理二进制文件会效果很糟糕，但是也是可以使用的。

标准输入、输出、错误流是三个特殊的文件，这个已经提到过多次了，它们的fd分贝是0,1,2。

## 标准IO
之前提到过一篇[系统级IO](http://wuxu92.github.io/system-level-io)。系统级IO（Unix IO)一般称为低级IO(low level io)，使用操作系统提供的基本IO服务，标准IO则是标准C提供的保准所有操作系统都可用的IO模型，ANSI C建立的标准IO模型具有很好的可移植性，我们讨论的标准IO就是它。

除了可以执行之外，标准IO相对于系统级IO还有两个优势：

1. 标准IO包含了很多专用的函数，可以方便地处理不同的IO问题。
2. 对于输入和输出进行了缓冲，也就是可以大块地转移信息，而不是每次一个字节地转移。

对于标准IO的学习主要就是一些标准IO函数的学习。这些函数本身不是很复杂，只需要大概记住它们的函数声明就可以使用了。这里不一一列举了，写一些比较常用的。

```
#include <stdio.h>

// 打开一个文件，返回一个新的流,
FILE * fopen(const char * restrict __filename, const char * resttrict __modes); 
// 打开一个文件，替换一个已存在的流
FILE *reopen(const char * restrict __filename, const char * restrict __modes, FILE * restrict __stream)
````
第二个参数的打开模式使用一个字符串表示，常用`r`, `w`, `a`, `r+`, `w+`, `a+`,注意**使用了w的模式都会删除原有文件的内容**。这几个模式都可以再加一个 b 表示使用二进制模式而非文本模式打开文件。不过对于Unix和Linux这样只有一种文件类型的系统，带b和不带b的模式是相同的。

可以看出`fopen`和系统级IO的`open`返回的不一样，`fopen`返回一个方便操作的文件指针fp，而`open`是返回一个int类型的fd。指针fp指向一个关于文件信息的数据包，而不是指向实际的文件。

```
int fgetc(FILE *);
int getc(FILE *);
int fputc(int, FILE *);

char * fgets(char * restrict __s, int __n, FILE * restrict __stream);
int fputs(const char * restrict, FILE * restrict);

int fclose(FILE *);	// 关闭成功返回0
int fflush(FILE *);	// 刷新文件

// code snippet
while ((ch = getc(fp) != EOF) {
	...
}
```
fgetc 和 getc 的主要区别是 getc是使用宏实现的，而fgetc是一个函数，这意味着一些区别：

1. getc的参数不能是一个有副作用的表达式（虽然没有语法错误，但是会很危险）
2. 可以获得`fgetc`的地址（函数指针）
3. 理论上来说，`getc` 速度更快，对于大量使用的情况，推荐使用 `getc`

文件的格式化输入输出：

```
int fprintf(FILE * restrict __stream, const char * restrict __format, ...);
int fscanf(FILE * restrict __stream, const char * restrict __format, ...);
```

文件指针操作：

```
int fseek(FILE * __stream, long int __off, int __whence);	// 设置位置指针为指定的值
long int ftell(FILE * __stream);		// 获取当前文件位置

int fgetpos(FILE * restrict __stream, fpos_t * restrict __pos);
int fsetpos(FILE * __stream, const fpos_t * __pos);
```
上面的`fseek`设定指针的位置是很常见的，其中第二的参数 offset 是一个可正可负可为零（为0用于测试该流是否允许设置指针位置）的long int 类型，第三个参数说明offset 的起始点。 ANSI 中定义了如下起始点常量：

- SEEK_SET 。 文件开始
- SEEK_CUR 。 当前指针
- SEEK_END 。 文件结尾

结合起始点与偏移量可以方便的设置到任意的文件位置。

fgetpos 和 fsetpos 使用一个 `fpos_t` 类型的指针作为设置值，可以用于更大的文件（超过long int范围长度的文件）。

## 标准IO内幕
哈哈哈

## 其他标准IO函数
列出来如下：

- `int ungetc(int c, FILE *fp)` 将指定字符c放回输入流
- `int feof(FILE *fp)` 判断是否到达文件结尾
- `int ferror(FILE *fp)` 测试错误指示器
- `void clearerr(FILE *)` 清楚文件结尾和错误指示器

```
int setvbuf(FILE * fp, char * buf, int mode, size_t size);
size_t fread(const void * ptr, size_t size, size_t nmemb, FILE * fp);
size_t fwrite(void *ptr, size_t size, size_t nmemb, FILE * fp);
```
servbuf可以建立一个供标准IO函数使用的替换缓冲区。（cpp 368）


完

参考：

- [http://stackoverflow.com/questions/18480982/getc-vs-fgetc-what-are-the-major-differences](http://stackoverflow.com/questions/18480982/getc-vs-fgetc-what-are-the-major-differences)
- C primer Plus
- CSAPP

