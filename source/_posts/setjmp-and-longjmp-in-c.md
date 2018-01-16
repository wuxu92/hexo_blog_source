---
date: 2015/12/23 12:34:50
title: 跳转 setjmp 和 longjmp
categories:
- programming
tags:
- C
- goto
---
虽然 goto 被认为是语言设计中的“毒瘤”，人人都避而远之，但是在真正的C开发者心中，goto不应该是让人害怕的东西。反而有一些比goto更加放肆的跳转函数！

## setjmp 和 longjmp
在C中，goto语句只能在函数内部进行跳转，而不能跨越函数，要执行这种跳转功能需要 `setjmp` 和  `longjmp` 配合使用，这两个函数对于处理发生在很深层嵌套函数调用中的出错情况是非常有用的，我们把它们成为**非局部跳转**，表示它们在栈上跳过多个调用栈帧，返回到当前函数调用路径上的某一个函数。

这两个函数的声明如下：

```
#include <setjmp.h>
int setjmp(jmp_buf env);
void longjmp(jmp_buf env, int val);
```
这两个函数让人感到比较奇怪的是，它的参数env看起来是一个结构体变量，而不是一个结构体指针，实际上是因为 jmp_buf 并不是结构体定义，而是某种形式的数组定义，本身就是一个指针，所以不需要使用取地址操作。

我们在希望返回的地方调用 `setjmp`，该调用会将用于恢复栈帧状态的信息保存到 env 变量中，因为 env 变量在 longjmp函数调用中要用到，而longjmp常在另一个函数调用，所以常需要**将env设置为全局变量**。

调用 setjmp 后可以在后续调用的函数中使用 longjmp 函数，它使用 setjmp 时所使用的 env 作为参数，第二个参数是一个非0值，则作为 setjmp 处恢复栈帧后的函数返回值，第二个参数可以用来作为多处调用 longjmp 返回到一个 setjmp时判断具体是那个 longjmp 的跳转。

```
#include <setjmp.h>
#include <stdio.h>
jmp_buf jbuf;
void handle(char *);
int main(void) {
    int i;
    char line[32];
    i = setjmp(jbuf);
    if (i != 0) puts("setjmp error");  // longjmp return comes here
    printf("i : %d\n", i);
    while(fgets(line, 32, stdin) != NULL) {
        handle(line);
    }
    puts("exit");
}
void handle(char *line) {
    if (line[0] == 'n' && line[1] == 'o') {
         longjmp(jbuf, 1);
    }
    printf("handle %s\n", line);
}
```

## 跳转中的变量
我们要考虑一下跳转后**自动变量**和**寄存器变量**（其他类型不需要考虑）的值的状态。当longjmp返回到main函数时，这些变量的值是否能恢复到以前调用setjmp是的值，或者这些变量的值保持为调用函数时的值？很遗憾，这个问题的答案是看情况，所有的标准称这些值是不确定的，虽然大多数的实现并**不回滚**这些自动变量和寄存器变量的值，但是在编译器使用优化策略时，其值可能会被回滚。

如果有一个自动变量，又要确保不回滚其值，可以定义其为具有 `volatile`属性，声明为全局变量或者静态变量的值在执行 longjmp 时保持不变（总是能保持最近所呈现的值）。

下面是一个示例，表示了不同类型的变量在 longjmp 跳转后值是否回滚：

```
#include <setjmp.h>
#include <stdio.h>

void f2(void);
void f3(void);
void f1(int, int, int, int);

jmp_buf jbuf;
int global;

int main(void) {
    int aut;
    register int reg;
    volatile int vola;
    global = 1; aut = 2; reg = 3; vola = 4;
    if (setjmp(jbuf) != 0) {
        printf("after long jmp\nglobal:%d, auto:%d, reg:%d, vola:%d\n",
                global, aut, reg, vola);
        exit(0);
    }
    global = 11; aut = 22; reg = 33; vola = 44;
    f1(global, aut, reg, vola);
}
void f1(int g, int a, int r, int v) {
    printf("in f1\ng:%d, a:%d, r:%d, v:%d\n", g, a, r, v);
    f2();
}
void f2(void) {
     f3();
}
void f3() {
     longjmp(jbuf, 1);
}
```

不使用优化的编译结果如下：

```
wuxu@centos7 % gcc -o jmpvar jmpvariable.c
wuxu@centos7 % ./jmpvar
in f1
g:11, a:22, r:33, v:44
after long jmp
global:11, auto:22, reg:33, vola:44
```
开启编译器优化的结果：

```
wuxu@centos7 % gcc -O2 -o jmpvarO2 jmpvariable.c
wuxu@centos7 % ./jmpvarO2
in f1
g:11, a:22, r:33, v:44
after long jmp
global:11, auto:2, reg:3, vola:44
```
可以看出，在启用编译器优化后，自动变量和寄存器变量都回滚了，而全局的，volatile修饰的变量咩有回滚值。

## 自动变量的潜在问题
对于自动变量，一个基本的规则是声明自动变量的函数已经返回后，不能再引用这些变量。下面是一个例子，使用一个自动变量作为文件流的缓冲区了：

```
#include <stdio.h>

FILE * open_file(const char *filename);

int main(int argc,  char *argv[]) {
    char *filename = "a.txt";
    FILE *fp = open_file(filename);
	// call some other function
    char buf[64];
    fread(buf, sizeof(char), sizeof buf, fp);
    puts(buf);
}


FILE *open_file(const char *filename) {
     FILE *fp;
     char databuf[BUFSIZ];
     if ((fp = fopen(filename, "r")) == NULL)
         puts("open file failed");

     if (setvbuf(fp, databuf, _IOLBF, BUFSIZ) != 0)  // IOLBF means line buffer io
         puts("setvbuf failed");

     return fp;
}
```
`open_file` 函数中，使用自动变量 databuf 的空间作为文件流的缓冲区，当函数返回时，databuf在栈上的的空间将由下一个被调用函数的栈帧使用，这个时候就产生了冲突（在实际测试（gcc 4.8.3, centos 7）中，并没有出现问题，文件流能正常读取）。不过按照理论这是容易出现问题，为了防止出现问题，应该使用全局存储空间静态地（static, extern）或者动态地(malloc等）为数组 databuf 分配空间。

基本完
