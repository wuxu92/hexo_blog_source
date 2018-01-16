---
title: APUE：信号
date: 2016-01-13 14:43:12
categories:
- programming
- c
tags:
- unix
- c
- linux
- programming
- signal
---
信号是软件中断，是比较底层的机制，这样一章断断续续看完了，但是很多地方都没有理解好。

信号提供了一种处理异步事件的方法，例如中断用户键入中断，会通过信号机制停止一个程序，或及早终止管道的下一个程序。信号机制在Unix的早起版本就提供了，不过早期的信号机制并不可靠，后来信号模型做了更改，增加了可靠信号机制（4.3 BSD、SVR3），POSIX.1对可靠信息号进行了标准化，现在的信号都基于这一标准。

## 概念
信号都有一个名字，信号的名字都是以 SIG 三个字符开头，信号在`<signal.h>`中定义为**正整数**常量（不同的系统可能定义在另外的头文件，不过那些头文件肯定会被 signal.h 包含），当然我们并不要去关心具体的定义是什么。

有多种方法向进程发送信号：

1. 中断按键，常见 `Ctrl+C` 发送中断信号（SIGINT）
2. 硬件异常产生信号，这是硬件检测到发送给内核的，如除数为0，无效的内存引用等
3. 进程调用 `kill` 函数可以将**任意信号**发送给进程/进程组，前提是接收信号的进程和发送信号进程所有者相同。
4. 用户在终端调用 `kill` 命令将信号发送给指定进程，常用来结束一个失控的后台进程， `kill -9 pid`
5. 当系统检测到某种**软件条件**已经发生，并应将其通知有关进程时也产生信号。
<!-- more -->
产生信号的事件对于进程来说是随机出现的，进程不能简单地测试一个变量来判断是否产生一个信号（可以测试 errno 变量判断是否产生错误）。而是必须告诉内核，当这个信号发生时，请执行下面的操作。这有和事件注册有点像，事先对信号注册处理函数，在信号到达时由内核自动调用该函数处理。

当信号到达是，可以告诉内核按下列的方式之一进行处理。

- 忽略该信号，大部分信号都是被忽略的，但有两种信号不能被忽略，`SIGKILL` 和 `SIGSTOP` 这两个信号要确保在任何条件下，内核和超级用户都能通过这两个信号终止或停止这个进程
- 捕捉信号，通知内核在某个信号到达时，执行一个用户定义函数。这是大部分信号相关应用要关心的。
- 执行默认动作。 每个信号都对应一个默认动作，对大多数信号系统的默认动作就是终止进程

常见/常用的几个信号:

1. SIGABRT		异常终止
2. SIGALRM		定时器超时，使用这个信号实现 sleep 函数
3. SIGCHLD		子进程状态改变，这在进程控制中使用过
4. SIGHUP		连接断开
5. SIGILL		非法硬件指令
6. SIGINT		终端中断符，这是最常用的信号，特别是不允许 Ctrl+c 终止运行时
7. SIGPOLL		可轮询事件（poll）
8. SIGSEGV		无效内存引用/段错误/segment violation
9. SIGWINCH		终端窗口大小改变/window change
10. SIGUSR1		用户定义信号1
11. SIGUSR2		用户定义信号2


## signal 函数
`signal`是Unix系统信号机制最简单也是最重要的接口。它的函数声明如下：

```c
#include <signal.h>
void (*signal(int signo, void (*func)(int)))(int);
```
这个声明有点复杂，要一点点分析：
首先，signal是一个函数指针，它的参数部分是`int, void (*func)(int)`,我们知道第二个参数是一个指向函数的指针，这个函数返回值是void，参数是一个int，也就是调用signal函数要传入两个参数，第一个是信号名（信号名是一个SIG开头的常量，实际定义为整形），第二个参数是一个函数，这个函数没有返回值且要有一个int参数。

再看signal的返回值，signal的返回值是一个函数地址，返回的函数使用一个int作为参数，返回的函数没有返回值。也就是调用signal函数将返回一个函数指针。这是第一次见到。

signal函数原型有点复杂，可以用下面的方式方便理解,这个定义和上面的signal定义是等价的：

```c
typedef void Sigfunc(int);
Sigfunc *singal(int Sigfunc *);
```

为什么signal要返回一个函数呢？这是因为使用signal为信号绑定了一个信号处理函数，那如果原来信号就有一个处理函数了呢？不能简单地丢掉它而要将它的引用返回给调用者，以防后面还需要它，**所以signal函数的返回值是指向在此之前的信号处理程序的指针**。这是很有用的，很多时候为一个信号绑定处理程序，过一段时间还要再恢复到原来的处理程序。

那如果绑定信号失败了呢？C语言中一般的调用失败反悔一个-1，但是 signal 调用返回的是一个函数指针，怎么通过一个函数指针判断是否绑定失败呢？还好系统定义了一个常量 `SIG_ERR` 只需要比较返回值和 `SIG_ERR`，如果它们相等则是signal调用失败。实际上 `SIG_ERR`常量也是一个函数指针，声明如下：

```c
#include <signal.h>
#define SIG_ERR (void (*)())-1
#define SIG_DFL (void (*)())0
#define SIG_IGN	(void (*)())1
```
另外两常量分别标识默认方法和忽略信号，即 default 和 ignore，它们可以用作 signal 方法的第二个参数。

signal的使用示例：

```c

#include <signal.h>
#include <unistd.h>
#include <stdio.h>

static void sig_user(int); // signal handler

int
main(void) {
  if (signal(SIGUSR1, sig_user) == SIG_ERR)
    printf("can't catch SIGUSR1\n");
  
  if (signal(SIGUSR2, sig_user) == SIG_ERR)
    printf("can't catch SIGUSR2\n");
  for (;;)
    pause();
}

static void
sig_user(int signo) {
  if (signo == SIGUSR1)
    printf("received sigusr1\n");
  else if (signo == SIGUSR2)
    printf("received sigusr2\n");
  else printf("received signal %d\n", signo);
}
```

当一个进程调用fork时，其子进程继承父进程的信号处理方式，因为子进程在开始时复制了父进程内存映像，所以信号捕捉函数地址在子进程是有意义的。

在早起的Unix系统（如V7）中，信号是不可靠的，这是指信号可能会丢失，一个信号发生了，但是进程却可能一直不知道这一点，同时进程对信号的控控制能力也很差，当然这都是早起Unix系统能够存在的问题，不需要太过关心。

## 可重入函数/Reentrant Functions
由于信号处理程序是异步进行的，这就牵涉到主进程函数调用和信号处理程序之间可能造成的冲突了。

进程捕捉到信号并对其进行处理时，进程正在执行的正常指令序列就被信号处理程序临时中断，它首先执行该信号处理程序中的指令，如果从信号处理程序反悔，则继续执行在捕捉到信号时进程正在执行的指令序列。但是在信号处理程序中，不能判断捕捉到信号时进程执行到何处，如果进程正在执行 malloc ，在其堆中分配另外的存储空间，此时捕捉到信号而插入执行信号处理程序，若在信号处理中又调用 malloc，那么返回进程时原来的地址分配很可能出错，还有向 getpwnam 这样的结果存放在静态存储单元的函数，如果在信号处理程序调用了的话也会导致原来的数据被覆盖。

SUS说明了信号处理程序中保证调用安全的函数，这些函数被称为**可重入的**，并被称为是异步信号安全的。除了可重入之外，在信号处理操作期间，它会阻塞任何引起不一致的信号发送。常见的可重入函数也挺多的，不过大部分函数还是**不可重入的**，一个可重入函数的列表如下：

| 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|
|abort | faccessat | linkat | select | socketpair|
|accept | fchmod | listen | sem_post | stat|
|access | fchmodat | lseek | send | symlink|
|aio_error | fchown | lstat | sendmsg | symlinkat|
|aio_return | fchownat | mkdir | sendto | tcdrain|
|aio_suspend | fcntl | mkdirat | setgid | tcflow|
|alarm | fdatasync | mkfifo | setpgid | tcflush|
|bind | fexecve | mkfifoat | setsid | tcgetattr|
|cfgetispeed | fork | mknod | setsockopt | tcgetpgrp|
|cfgetospeed | fstat | mknodat | setuid | tcsendbreak|
|cfsetispeed | fstatat | open | shutdown | tcsetattr|
|cfsetospeed | fsync | openat | sigaction | tcsetpgrp|
|chdir | ftruncate | pause | sigaddset | time|
|chmod | futimens | pipe | sigdelset | timer_getoverrun|
|chown | getegid | poll | sigemptyset | timer_gettime|
|clock_gettime | geteuid | posix_trace_event | sigfillset | timer_settime |
|close | getgid | pselect | sigismember | times|
|connect | getgroups | raise | signal | umask|
|creat | getpeername | read | sigpause | uname|
|dup | getpgrp | readlink | sigpending | unlink|
|dup2 | getpid | readlinkat | sigprocmask | unlinkat|
|execl | getppid | recv | sigqueue | utime|
|execle | getsockname | recvfrom | sigset | utimensat|
|execv | getsockopt | recvmsg | sigsuspend | utimes|
|execve | getuid | rename | sleep | wait|
|_Exit | kill | renameat | sockatmark | waitpid|
|_exit | link | rmdir | socket | write|

不可重入函数的特点是：

- 已知它们使用静态数据结构，如 `getpwnam`
- 它们调用 malloc 或 free，因为这些函数为它所分配的存储区维护了一个链表，插入信号处理程序可能修改这个链表。
- 它们是标准I/O函数，标准IO函数的很多实现都以不可冲入方式使用全局数据结构。虽然有时在信号处理程序中使用 printf ，但这是不可靠的，这样信号处理程序可能中断主进程的 printf 函数调用。

还有一个要注意的就是 `errno`，我们知道它是定义在 `<error.h>`中的全局变量，用于说明前一个调用是否有错误，每个线程只有一个 errno，所以即使信号处理程序调用上面的可重入函数也可能会修改errno原值，常见的做法是在信号处理程序调用上表中的函数时，先保存errno，在调用后再恢复errno。

## 信号屏蔽字
每个进程都有一个信号屏蔽字（signal mask）,就像文件创建权限屏蔽字。信号屏蔽字规定了当前要阻塞递送到进程的信号集合。回忆一下文件创建屏蔽字，用屏蔽字的一位代表一个权限，信号屏蔽字也是相似的思路，不过信号编号很多，可能会超过一个整形所包含的二进制位数，因此不能简单用一个整形表示。POSIX.1定义了一个数据类型 `sigset_t` 它可以容纳一个**信号集**。这个类型一般定义在 <bit/sigset.h> 中，这个头文件被 <signal.h> 包含，所以也不需要单独在包含了。

关于信号集，和一些常用的操作函数声明如下：

```c
#include <signal.h>
int sigemptyset(sigset_t *set);		# 清空信号集
int sigfillset(sigset_t *set);		# 添加所有的信号到信号集
int sigaddset(sigset_t *set, int signo);	# 向信号集添加一个信号
int sigdelset(sigset_t *set, int signo);	# 从信号集删除一个信号
int sigismember(sigset_t *set, int signo);	# 判断某个信号是否在信号集中
```
在应用程序使用信号集之前，都要对该信号集调用一次 `sigemptyset` 或者 `sigfillset`，因为C编译程序将不赋初值的外部变量和静态变量都初始化为0，而这与是否与个顶的信号集实现相对应是不清楚的。

## 函数： kill 和 raised
可以通过 `kill` 函数将信号发送给进程或者进程组，`raised`则允许向自身发送信号。

```c
#include <signal.h>
int kill(pid_t pid, int signo);
int raise(int signo);
```
pid可正可负可为0与-1，其意义与之前使用 pid_t 类型参数的方法一样，为负时表示进程组，0标识所有同一进程组的，-1标识发送给所有有权限的进程。

POSIX.1 将信号编号为0定义为**空信号**，如果signo为0，则kill仍执行正常的错误检查，但不发送信号，这常被用来**确定一个特定的进程是否仍然存在**。如果像一个不存在进程发送空信号，kill返回-1，errno设置为 `ESRCH`。


## 函数 alarm 和 pause
`SIGALRM`是常用的信号，alarm函数可以设置一个定时器，在将来的某个时刻该定时器会超时，当定时器超时时，产生 `SIGALRM` 信号，如果忽略不捕捉这个信号，默认动作是终止调用该 alarm函数的进程。大部分使用 `SIGALRM`信号的进程都会捕捉该信号。

```c
#include <signal.h>
unsigned int alarm(unsigned int seconds);
```

每个进程只有一个闹钟时间，如果在alarm时，之前已经注册的闹钟事件还没有超时，则该alarm调用的返回值为之前闹钟的剩余时间。

另一个 `pause`函数使调用进程挂起知道捕捉到一个信号。

```
#include <signal.h>
int pause(void);
```
只有执行了一个信号处理程序并从其返回时，pause才返回，这种情况下，pause返回-1，errno设置为 `EINTR`。

## 设置信号屏蔽字 sigprocmask
进程的信号屏蔽字规定了当前阻塞而不能递送给该进程的信号集。调用函数 sigprocmask 可以检测或更改，或同时进行检测和更改信号屏蔽字。其函数声明如下：

```
#include <signal.h>
int sigprocmask(int how, const sigset_t *restrict set, const sigset_t *restrict oldset);
```
如果oldset是非空指针，则将进程的当前信号屏蔽字保存到oldset中。
如果set是一个非空指针，则参数**how指示如何修改当前屏蔽字**

how有三个可选操作，它们是三个定义常量：

- SIG_BLOCK		该进程新的屏蔽字是当前屏蔽字与set的**并集**，set中要包含希望阻塞的附加信号
- SIG_UNBLOCK	新的屏蔽字是当前屏蔽字和set信号集**补集的交集**，所以set中要包含希望解除屏蔽的信号
- SIG_SETMASK	新的屏蔽字是 set 指向的值

如果set是空指针，则不改变什么。sigprocmask是为单线程定义的，在多线程中使用另一个函数。

### sigpending
`sigpending` 函数返回一个信号集，对于调用进程而言，返回的信号集中的各信号是阻塞不能递送的，同set参数返回。

```c
#include <signal.h>
int sigpendding(sigset_t *set);
```
这里有一个阻塞信号是否排队的问题，即一个信号在阻塞期间多次触发，系统是否将这些信号排队起来还是只保存最近的一个，大部分系统的实现只并不排队，只保存最近的一个信号。

### sigaction
`sigaction`函数的功能是检查或修改于指定信号相关联的处理动作，此函数取代了Unix早起使用的`signal`函数，现代的`signal`函数实现中常使用 `sigaction`。这个函数比较复杂，其声明如下：

```c
#include <signal.h>
int sigaction(int signo, const struct sigaction *restrict act, struct sigaction *restrict oact);
```
这里的第二和第三个参数是 sigaction 结构体，注意结构体与函数重名是可以的，它们并不发生冲突。该结构体定义如下：

```c
struct sigaction {
  void (*sa_handler)(int);	// addr of signal handlerot SIG_DFL/SIG_IGN
  sigset_t sa_mask;			// additional signals to block
  int sa_flags;				// signal options
  void (*sa_sigaction)(int, siginfo_t *, void *)	// alternate handler
}

struct siginfo {
  int si_signo;
  int si_errno;
  int si_code;
  ..
  int si_status;
  /** possibly other fields */
}
```
这两个结构体有点复杂。需要使用时再具体分析吧。Chapter 10.6 page 278


后面还有一些没看懂的，之后再补吧。

暂时完 