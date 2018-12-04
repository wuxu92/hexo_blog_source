---
date: 2015/12/21 12:34:50
title: APUE：系统数据文件和信息
categories:
- programming
tags:
- APUE
- Unix
- Linux
- C
---
![](/images/Liux-logo.png)
Unix系统的正常运作需要使用大量与系统有关的数据文件，例如口令文件 /etc/passwd 和组文件 /etc/group 就经常被多个程序使用，由于历史原因，这些文件都是ASCII文本文件，并且使用标准IO库读这些文件...但是，对于较大的系统，顺序扫描指令文件很花费时间，我们需要能够以非ASCII文本格式存储这些文件，但仍然向使用其他文件格式的应用程序提供接口。

## 口令文件
口令文件就是 /etc/passwd 文件，里面记录了系统的用户的信息，包括用户名，密码（可能单独存放在另一个文件），备注，家目录，默认shell等信息，对Linux比较熟悉的话，对这个文件应该也不陌生。不同的系统的passwd 文件实现不同，Linux实现了7个字段，而FreeBSD 和 MacOS X则实现了全部10个字段（至于各个字段是什么...其实并不重要）。

与口令文件操作相关的函数和结构声明包含在 <pwd.h> 中，其中定义的结构体 `struct passwd ` 用来保存一个用户对应的信息。

```
#include <pwd.h>
struct passwd
{
  char *pw_name;		/* Username.  */
  char *pw_passwd;		/* Password.  */
  __uid_t pw_uid;		/* User ID.  */
  __gid_t pw_gid;		/* Group ID.  */
  char *pw_gecos;		/* Real name.  */
  char *pw_dir;			/* Home directory.  */
  char *pw_shell;		/* Shell program.  */
};

struct passwd *getpwuid(uid_t uid);
struct passwd *getpwdnam(const char *name);
```
上面两个函数分别通过uid和username获取口令文件中的信息，它们都返回一个指向passwd结构的指针，该结构在执行函数时填写了信息，一般返回的passwd结构是**函数内部的静态变量**，只要调用任意相关函数，其内容就会被重写。
<!-- more -->
其他还是有 `getpwent(), setpwent(), endpwent()` 函数不太会用到，它们都是SUS(Single Unix Specification)中定义为 XSI 扩展的，没有在 POSIX.1中定义，不过现在的系统大都实现了这些函数。它们被用来读取整个口令文件，如果需要在查找手册吧。

> 这些函数定义的 ent 后缀表示 entry，即一个表项，这是一个约定的命名习惯。

## 阴影文件/shadow passwords
阴影文件即用户密码单向加密后的用户口令副本，对于一个加密口令，没有算法能将其反变换到明文口令，但是可以对口令进行猜测，为了使其图这样做的人难以获得原始资料（加密口令），现在的系统将加密口令存放在一个称为阴影口令的文件中。该文件至少要包含用户名和加密口令，与口令相关的其他信息也可存放在该文件中。

与阴影文件操作相关的结构体为 `spwd`(shadow password)。阴影口令文件不是一般用户可读的，只有少数的程序可以访问加密口令，如 login，passwd。因为阴影口令的存在使得普通口令文件允许普通用户自由读取。

阴影口令操作包含在 <shaodw.h> 头文件中。

```
struct spwd
{
    char *sp_namp;		/* Login name.  */
    char *sp_pwdp;		/* Encrypted password.  */
    long int sp_lstchg;		/* Date of last change.  */
    long int sp_min;		/* Minimum number of days between changes.  */
    long int sp_max;		/* Maximum number of days between changes.  */
    long int sp_warn;		/* Number of days to warn user to change the password.  */
    long int sp_inact;		/* Number of days the account may be inactive.  */
    long int sp_expire;		/* Number of days since 1970-01-01 until account expires.  */
    unsigned long int sp_flag;	/* Reserved.  */
};
struct spwd *getspnam(const char *name);
struct spwd *getspent(void);
void setspent(void);
void endspent(void);
```
其使用与普通口令文件操作类似。

## 组文件/etc/group
组文件指 /etc/group 文件，里面保存了系统用户组相关的信息，相关的函数包含在头文件 <grp.h> 中，定义了一个 group 结构，用于保存一个组信息。

```
struct group
{
    char *gr_name;		/* Group name.	*/
    char *gr_passwd;		/* Password.	*/
    __gid_t gr_gid;		/* Group ID.	*/
    char **gr_mem;		/* Member list.	*/
};
```
其中的 gr_mem 成员是一个字符转数组的指针，用于保存该组的多个成员。其中指向组的加密口令（ge_passwd）并不是 POSIX.1 要求的，不过大部分系统都实现了这个字段，一般不用。

相关的函数：

```
#include <grp.h>
struct group *getgrgid(gid_t gid);
struct group *getgrnam(const char *name);
struct group *getgrent(void);
void setgrent(void);
void endgrent(void);
```

## 附属组和其他数据文件
跳跳跳

## 系统表示/uname
在Linux系统中，常通过 `uname -a` 命令查看系统的版本和编译信息。POSIX.1定义了相关的结构体和函数uname。下面是相关结构体（部分）和函数。

```
#include <sys/utsname.h>
struct utsname
  {
    char sysname[];	/* Name of the operating system.  */
    
    char nodename[];	/* Name of this node on the network.  */
    char release[];		/* Current release level of this implementation.  */
    char version[];		/* Current version level of this release.  */
    char machine[];		/* Name of the hardware type the system is running on.  */
};
int uname(struct utsname *name);
```
上面的utsname结构只是Linux实现的主要部分，省略了数组长度，具体可以通过 `/usr/include/sys/utsname.h` 文件查看，大概在48行。

在 <unistd.h> 有一个 `gethostname()`函数用于返回主机名，也是很常用的：

```
#include <unistd.h>
int gethostname(char *name, int namelen);	// namelen 指定最大长度为 HOST_NAME_MAX
```
主机名通常在系统自举（bootstrap）时由/etc/rc 或 init 取自一个启动文件完成设置。

## Unix时间
由Unix内核提供的基本时间服务是计算自协调世界时（UTC: Coordinated Universal Time）公元1970年1月1日 00:00:00 这一时间点到现在经过的秒数。这种秒数以数据类型 `time_t` 表示，我们称它们为**日历时间**。日历时间包括时间和日期。

```
#include <time.h>
time_t time(time_t *calptr); // 出错返回-1
```
我们知道 time_t 是使用typedef定义的基础类型，至于其具体的数据类型是什么，C语言并没有规定，只需要它是数字类型即可，现在系统一般使用 unsigned integer 类型，可能是32为或者64位的。所以实际使用time返回的就是Unix 时间戳（timestamp）。

另一个重要的结构体是 `timespec` ，其定义如下，提供sec和nanoseconds成员。

```
#include <time.h>
struct timespec
{
    __time_t tv_sec;		/* Seconds.  */
    __syscall_slong_t tv_nsec;	/* Nanoseconds.  */
};
int clock_gettime(clockid_t clock_id, struct timespec *tsp);
```
其中的 __time_t 一般是 time_t 的底层类型，即常有 `typedef __time_t time_t`。

clock_gettime函数应该是包含在头文件 <time.h> 中，但是APUE书本上写的是 <sys/time.h>，应该是一个笔误。其中的clockid_id 表示系统时钟的类型，POSIX.1扩展增加了多个系统时钟的支持，使用宏定义表示。

| 标识符 | 说明 |
|---|---|
| CLOCK_REALTIME  | 实时系统时间 |
| CLOCK_MONOTONIC | 不带负跳数的实时系统时间 |
| CLOCK_PROCESS_CPUTIME_ID | 调用进程的CPU时间 |
| CLOCK_THREAD_CPUTIME_ID  | 调用线程的CPU时间 |

常用 CLOCK_REALTIME获得比 time() 更精确的时间（带 nanoseconds）。

时间存在多种表示形式，各个时间函数之间的关系如下，这**张图超级有用**，其中方框中的是结构体，箭头上的是函数。
![](/images/Liux/time-functions.png)

其中的几个函数声明如下：

```
#include <time.h>

struct tm
{
  int tm_sec;			/* Seconds.	[0-60] (1 leap second) 可能闰秒*/
  int tm_min;			/* Minutes.	[0-59] */
  int tm_hour;			/* Hours.	[0-23] */
  int tm_mday;			/* Day.		[1-31] */
  int tm_mon;			/* Month.	[0-11] */
  int tm_year;			/* Year	- 1900.  */
  int tm_wday;			/* Day of week.	[0-6] */
  int tm_yday;			/* Days in year.[0-365]	*/
  int tm_isdst;			/* DST.		[-1/0/1]*/
};

struct tm *gmtime(const time_t *calptr);
struct tm *localtime(const time_t *calptr);
time_t mktime(struct tm *tmptr);
```
tm常被称为分解时间。localtime 和 gmtime 的区别是 localtime 将日历时间转换成本地时间（考虑本地时区和夏令时标志），而 gmtime 则将日历时间转换成协调统一时间。

实际使用中常需要将时间戳或者时间的结构体用易读的字符串打印出来，这就要使用 strftime strftime_l 函数，这两个函数是类似于printf的时间值函数，它们比较复杂，通过多个参数**定制产生的字符串**。

```
#include <time.h>
size_t strftime(char *restrict buf, size_t maxsize,
	const char *restrict format,
	const struct tm *restrict tmptr);

size_t strftime_l(char *restrict buf, size_t maxsize,
	const char *restrict format,
	cosnt struct tm *restrict tmptr, locale_t locale);
```
strftime 和 strftime_l 的区别是后者可以通过 locale参数设定区域，而前者使用 TZ(timezone) 环境变量指定区域。

这两个函数签名比较复杂，其实理解不难，就是将 tmptr 指向的分解时间结构的内容使用 format 指定的格式生成字符串保存到 buf 指向的字符串中。

还有两个函数， `asctime` 和 `ctime` 能产生指定格式的字符串，类似于 date 命令的输出，使用很简单，不过它们都被标识为弃用了，因为它们容易受到缓冲区溢出问题的影响。

format指定的格式和printf类似，不过有更多的特定字符，而且没有宽度修饰符。输出格式转换说明表如下：

![](/images/Liux/strftime-format.png)

其中大部分都比较容易理解，其中 %U, %V, %W 都与周数有关，它们之间有细微的区别， %U的周数是该年中第一个星期日的周为第一周，， %W是该年第一个星期一的周为第一周，%V则是包含1月1日的那一周包含了新一年的4天或者更多，那么这周是这年的第一周，否则认为是上一年的最后一周。

一个示例： `strftime(strtime, 36, "%a %b %d %H:%M:%S %Z %z %Y", localtime(&t));` 输出为 `Mon Dec 21 15:39:44 CST +0800 2015`。

有了将时间字符串表示，当然还需要将一个符合时间表示的字符串转换成一个系统可以理解的时间值的需求。C提供了指定format的字符串时间转换函数：

```
#include <time.h>
char *strptime(const char *restrict buf, const char *restrict format,
	struct tm *restrict tmptr);
```
format参数给出了buf参数指向的缓冲区内的字符串的格式，虽然与strftime函数的格式稍有不同，但是基本类似。

![](/images/Liux/strptime.png)

时间相关的函数很多都是用 TZ 环境变量的值，Linux可以用 tzselect 工具设定tz，这是一个交互性的文字界面工具。

基本完成

参考：

- [http://Liux.die.net/man/3/clock_gettime](http://linux.die.net/man/3/clock_gettime)
- [http://en.cppreference.com/w/c/chrono/time_t](http://en.cppreference.com/w/c/chrono/time_t)
- [http://stackoverflow.com/questions/471248/what-is-ultimately-a-time-t-typedef-to](http://stackoverflow.com/questions/471248/what-is-ultimately-a-time-t-typedef-to)
- Unix 环境高级编程
