---
date: 2015/12/15 12:34:50
title: APUE：文件和目录
categories:
- programming
tags:
- APUE
- C
- Unix
- Linux
---
![](/images/Liux-logo.png)
前面已近有一篇系统级IO，那是 CSAPP 中的一章，讲了Unix IO的内容，这一次是Unix环境高级编程的一章，也是讲Unix/Linux下的文件IO，可能这里讲编程的内容多一些。

我们知道通过 low level 的 open, creat 方法调用可以获得一个 file description ,它是一个整数，对应到进程文件描述符表的一项，使用这个整数就可以标识这个打开的文件。

## 文件信息
对于一个打开的文件的 fd， 我们已经知道有一个 `stat` 函数可以获得该文件的信息，其实有另外好几个类似的函数（在64位系统还是有stat64的一套函数），其函数声明如下：

```
#include <sys/stat.h>
int stat(const char *restrict pathname, struct stat *restrict buf);
int fstat(int fd, struct stat *buf);
int lstat(const char *restrict pathname, struct stat *restrict buf);
int fstatat(int fd, const char *restrict pathname, struct stat *restrict buf, int flag);
```
它们的作用都是获得某个文件的相关信息，文件信息被保存在一个结构体中，上面的函数调用会填充这个函数体的成员字段。stat和fstat比较好理解，只是第一个参数不同（不考虑参数的修饰符），stat使用一个文件名做参数，fstat使用一个fd做参数，实际的效果是一样的，不过stat的buf参数使用了restrict修饰符，这个修饰符在之前的文章中已经解释过了，它提示编译器保证该指针指向的对象不被当前指针以外的指针修改。
<!-- more -->
lstat的作用是当参数的文件是一个符号链接时，lstat返回该符号链接有关的信息（符号链接有自己的信息），而不是符号链接指向的文件的信息。`fstatat`函数为一个相对于打开目录（fd指向）的路径名返回文件信息，flag参数使用一个预定义常量，控制当参数是一个符号链接时是否跟随该链接（follow），默认返回链接指向的文件的信息，设置为 `AT_SYMLINK_NOFOLLOW` 使返回链接本身的信息。如果fd参数的值是 `AT_FDCWD` 并且pathname是一个相对路径则计算相对于当前目录的pathname，如果pathname是一个绝对路径则忽略fd参数。

让我们先看下这个结构体 `struct stat`:

```
struct stat {
	mode_t		st_mode;
	ino_t		st_ino;
	dev_t		st_dev;
	dev_t		st_rdev;
	nlink_t		st_nlink;
	uid_t		st_uid;
	gid_t		st_gid;
	off_t		st_size;
	struct timespec	st_atime;
	struct timespec	st_mtime;
	struct timespec	st_ctime;
	blksize_t	st_blksize;
	blkcnt_t	st_blocks;
}
```
其中各成员基本对应到Linux文件的一项属性。这个结构体在不同实现的定义可能有所不同，比如POSIX.1中并未要求 st_rdev, st_blksize, st_blocks 等字段，而SUS XSI 扩展定义了这些字段。虽然有一些区别，但是基本形式是差不多的。

这个结构体非常重要，我们经常要用到的字段包括 st_mode， st_uid, st_gid等，要理解其中一些常用成员的用法。

## 文件类型
我们知道Unix下有普通文件和目录两种类型，但是还有另外一些特殊文件类型，下面简单介绍：

1. 普通文件，最常见的文件类型，对于Unix内核而言，文本和二进制数据并无区别，对普通文件内容的解释由处理该文件的应用程序进行。对于ELF文件则是内核可理解的二进制形式。
2. 目录文件，这个也比较了解了，目录文件包含了其他文件的名字以及指向这些文件有关信息的指针。
3. 块特殊文件，提供对设备带缓冲的访问，每次访问以固定长度为单位。
4. 字符特殊文件，提供对设备不带缓冲的访问，每次访问长度可变，系统中的设备要么是字符特殊文件，要么是块特殊文件。
5. FIFO，这种特殊的文件用于进程间的通信，也称为命名管道。
6. 套接字，socket这个也比较熟悉了。
7. 符号链接，这中我呢间指向另一个文件

文件类型的信息保存在 stat结构体的 st_mode 成员中，通过预定义的类函数宏判断文件类型，这些宏的名称为 `S_ISXXX()` 将stat.st_mode 作为宏参数，XXX为文件类型，包括： REG, DIR, CHR, BLK, FIFLE, SOCK, LNK 分别对应上面的其中类型。这些宏定义在 <sys/stat.h> 中。

## ID和Group
当程序要打开或者读写一个文件时，首先要对其进行权限检查，要看进程是否有读写一个文件的权限主要靠文件相关的id和进程相关的id的关系以及文件的访问权限设置。

与一个进程关联的id有多个，常见的6个位：实际用户ID，实际组ID，有效用户ID，有效组ID，附属组ID，保存的设置用户ID，保存的设置组ID。实际ID表明我们究竟是谁，有效ID决定文件访问权限，保存的ID有exec函数保存，在执行一个程序时，包含了有效用户ID和有效组ID的副本。

通常有效用户ID等于实际用户ID，有效组ID等于实际组ID。

这里还有一个设置用户ID和设置组ID的话题，表示可以在执行文件的有效用户id设置为文件所有者的id，这通过设置 stat.st_mode 中的一个标志实现，比如 passwd 命令可以不用sudo就修改当前用户的密码，但是这个命令要修改 /etc/passwd 文件是需要root访问权限，通过设置用户id将执行进程的有效用户id设置为 /etc/passwd 的所有者id（即root）得到额外的权限。因为设置用户ID程序会获得额外的权限，因此也变得**更加危险**，编写这种程序时要特别小心。

熟悉Linux的对Linux的文件权限设置比较熟悉了，777，755，600等，就不细说了。要注意理解的是目录的可执行权限与目录的可读权限的区别。文件访问权限的9个访问权限位，分为3类分别设置了一个宏定义：

```
S_IRUSR
S_IWUSR
S_IXUSR

S_IRGRP
S_IWGRP
S_IXGRP

S_IROTH
S_IWOTH
S_IXOTH
```

新建文件和目录的用户ID设置为进程的有效用户ID，而组ID有两种选择：1.进程的有效组ID，2.其所在目录的组ID。

## 权限

### umask
umask为进程设置**文件模式创建屏蔽字**（file mode creation mask），并返回之前的值（这是一个没有出错返回的函数）。

```
#include <sys/stat.h>
mode_t umask(mode_t cmask);

umask(0);	// 不屏蔽权限位
umask(S_IXUSR|S_IXGRP|S_IXOTH);		// 屏蔽所有的执行位
```
其中cmask是上面列出的9个权限常量中的**若干个按位或**构成的。比如 `S_IRUSR | S_IRFRP | S_IROTH`。在创建一个文件或目录时，一定会使用文件模式创建屏蔽字。

记得在运行时修改umask的值，很多程序员会忘忘记修改它，通常在登录时由shell的启动文件设置一次屏蔽值，为了确保指定的权限位已激活，要在运行时修改umask值。否则已有的umask值可能会关闭一些权限位。


### chmod, fchmod, fchmodat
使用过Linux的用户对chmod命令很定不陌生，它用来修改已有文件和目录的权限，使用非常方便， `chmod 777 ****` 就可以将一个文件权限全打开，在C中可以用上面三个函数实现修改权限。

```
#include <sys/stat.h>
int chmod(const char *pathname, mode_t mode);
int fchmod(int fd, mode_t mode);
int fchmodat(int fd, const char *pathname, mode_t mode, int flag);
```
这三个函数参数的意义和前面 stat 系列函数是一样理解的。这里设置的mode_t 的取值较前面的要多些，可以设置 执行时设置用户ID(`S_ISUID`)，执行时设置组ID(`S_ISGID`)，保存正文/粘着位(`S_ISVTX`)三个宏。另外有更方便的同时设置可读可写可执行的三个宏： `S_IRWXU`, `S_IRWXG`， `S_IRWXO`，分别对应到用户、组、其它的读写执行。

类似有修改用户id，组id的函数可以调用，它们在 unistd.h 中声明，其函数声明如下：

```
#include <unistd.h>
int chown(const char *pathname, uid_t owner, gid_t group);
int fchown(int fd, uid_t owner, gid_t group);
int fchownat(int fd, const char *pathname, uid_t owner, gid_t group);
int lchown(const char *pathname, uid_t owner, gid_t group);
```
这些函数用的不多，具体细节需要的时候再查手册把。

### 粘着位
前面提到了**粘着位**，熟悉Linux的用户应该听过这个名词，在 /tmp 和 /vay/tmp 目录的介绍中肯定会介绍这个特殊的权限位。粘着一词的由来是在Unix系统还未使用请求分页技术的早期保本中为了保证可执行程序能快速地载入内存而设置的（通过将程序正文的副本保存在交换区），后来的Unix版本称之为 保存正文位（saved-text bit) 因此常量定义为 `S_ISVTX`。

现在的系统扩展了粘着位，如果对一个目录设置了粘着位，只有对该目录具有写权限的用户**并且**满足下列条件之一的才能删除或者重命名该目录下文件：

- 拥有此文件
- 拥有此目录
- 是超级用户

这种对删除/重命名操作的特殊权限要求特别符合 /tmp 和 /var/tmp 目录的要求，任何用户都可以在这两个目录创建文件，其他用户都有读写执行权限，但是**用户不能删除或重命名其他用户的文件**。

## 文件长度
stat结构中的 st_size 成员表示以字节为单位的文件的长度，此字段值对普通文件，目录文件和符号链接有意义。对于符号链接，其长度是在文件名中的实际字节数，例如`lib->/usr/lib`，这个符号链接的长度为7。

现在的操作系统大部分都提供了 stat 结构的 st_blksize 字段和 st_blocks 字段，第一个确认文件IO较合适的块长度，第二个是所分配的实际512字节块的块数。当使用 st_blksize 用于读操作时，读取一个文件所需要的时间最少，为了提高效率，标注IO库也试图一次读写一个 st_blksize 个字节。

### 空洞
之前的文章介绍过文件空洞的存在，空洞是所设置的偏移量超过文件尾端并写入数据后造成的。空洞并不会占用实际的磁盘空间，但是使用ls查看文件大小时会计算空洞的大小，文件系统使用了若干块以存放指向实际数据块的各个指针。

### 文件截断
unistd.h 中有一个用于截断一个文件的函数，truncate和ftruncate，它们的声明如下：

```
#include <unistd.h>
int truncate(const char *pathname, off_t length);
int ftruncate(int fd, off_t length);
```
这个很容易理解，就是将文件长度阶截断为length长。如果length超过文件长度，文件长度将增加，并填充0。另使用open函数时，指定 O_TRUNC 标志可以截断文件长度为0。

## 文件系统
Unix/Linux系统下可以使用多种文件系统实现，如传统的基于BSD的Unix文件系统（UFS），主要是要了解i节点和指向i节点的目录项之间的区别。

i节点，目录快，数据块，文件，i节点编号，i节点数组等。i节点包含了文件有关的所有信息，文件类型，文件访问权限位，文件长度和指向文件数据块的指针等。stat结构中的大多数信息都取自i节点。只有两项数据存放在目录项中：文件名和i节点编号。为文件重命名或者移动文件时，也只是构造一个指向享有i节点的新目录项，并删除旧的目录项即可。

下面这个图是创建一个目录testdir后的文件系统示例：
![](/images/post/apue-inodes.png)
其他的就不具体介绍了，文件系统比较复杂。理解inode之后基本就可以了。

## 链接函数
任何一个文件可以有多个目录项指向其i节点，创建一个指向现有文件的链接方法是使用link函数或linkat，类似，取消一个指向现有文件的链接使用 unlink 和 unlinkat 函数。它们包含在 <unistd.h> 中，函数声明如下：

```
#include <unistd.h>
int link(const char *existringpath, const char *newpath);
int linkat(int efd, const char *existingpath, int nfd, const char *newpath, int flag);
int unlink(const char *pathname);
int unlinkat(int fd, const char *pathname, int flag);
```
其中的 ××at 函数和之前的类似明明方式的意义相同。如果linkat的两个文件描述符参数的任意一个设置为 AT_FDCWD 那么相应的路径名（如果它的相对路径）就通过相对于当前目录进行计算，如果任意路径是绝对路径，相应的文件描述符参数就会被**忽略**。

unlink函数删除指定的目录项，并将由pathname所引用文件的链接计数减1.如果还有对该文件的其他链接，则仍可以通过其它链接访问该文件的数据，当链接计数达到0时，文件内容才会被删除。另外只要有进程打开了这个文件也不能删除文件的内容，内核会首先检查打开该文件的进程个数，这个计数达到0再检查其链接计数，如果也是0就可以删除文件的内容。

unlink 的特性常用来确保即使程序奔溃时，它所创建的**临时文件**也不会遗留下来，进程用 `open` 或 `creat` 创建一个文件，然后**立即调用**`unlink`，因为该文件仍旧是打开的，所以不会将其内容删除。只有当进程关闭时或终止时，该文件的内容才会被删除。

重命名函数：

```
#include <stdio.h>
int rename(const char *oldname, const char *newname);
int renameat(int oldfd, const char *oldname, int newfd, const char *newname);
```
如果oldname是一个文件，newname已经存在，则newname不能是目录项，如果newname也是文件则删除newname的文件再将oldname重命名为newname。如果oldname是一个目录且newname已经存在，那么如果newname是一个空目录，则可以将其删除，然后将oldname目录重命名我newname。

另外对于目录重命名，newname不能包含oldname作为路径前缀，即不能将 `/var/foo` 重命名为 `/var/foo/bar`。不能对 `.` 和 `..` 进行重命名。

## 符号链接
符号链接是对一个文件的间接指针，它与上一节的硬链接有所不同。硬链接直接指向文件的inode，符号链接是为了避开硬链接的一些限制：

- 硬链接通常要求链接和文件位置在同一文件系统中
- 只有超级用户才能创建指向目录的硬链接（在文件系统支持的条件下）

符号链接，也常称为软连接，没有上面的限制。符号链接和系统命令 `ln -s target link_name` 的效果是一样的，硬链接则和 `ln target link_name`类似。

创建符号链接的函数声明如下：

```
#include <unistd.h>
int symlink(const char *actualpath, const char *sympath);
int symlinkat(const char *actualpath, int fd, const char *sympath);
```
注意的是，创建符号链接时，并不会检查 actualpath 是否已经存在。 `open` 函数打开一个符号文件时会跟随（follow）符号链接，所以要打开符号链接本身需要使用 readlink 函数。

```
#include <unistd.h>
ssize_t readlink(const char *restrict pathname, char *restrict buf, sie_t bufsize);
ssize_t readlinkat(int fd, const char *restrict pathnae, char *restrict buf, size_t bufsize);
```
这两个函数组合了 open, read, close 的所有操作。返回读取到buf中的字节数。

**关于文件时间的读取、修改暂时跳过**

## 目录
目录操作就是创建目录，删除目录，重命名，读取目录的函数，需要注意的是这些函数包含在不同的头文件中，有 <sys/stat.h>, <unistd.h>, <dirent.h>等头文件中。

```
#include <sys/stat.h>
int mkdir(const char *pathname, mode_t mode);
int mkdirat(int fd, const char *pathame, mode_t mode);
```
创建目录指定文件访问权限mode由进程的文件模式创建屏蔽字修改。对于目录通常至少要设置一个执行权限位，以允许访问该目录中的文件名。

```
#include <unistd.h>
int rmdir(const char *pathname);
```

### 读取目录
对某个目录具有访问权限的任意用户都可以读该目录，但是为了防止文件系统混乱，只有内核才能写目录。一个目录的写权限位与执行权限位决定在该目录能否创建新文件以及删除文件，并不表示能否写**目录本身**。

```
#include <dirent.h>
DIR *opendir(const char *pathname);
DIR *fdopendir(int fd);

struct dirent *readdir(DIR *dp);

void rewinddir(DIR *dp);
int closedir(DIR *dp);
long telldir(DIR *dp);
void seekdir(DIR *dp, long loc);
```
其中前两个函数用于打开一个目录，返回一个 DIR 指针，后面的函数对这个指针进行操作，其中最常用的 `readdir` 用于从开打的目录读取一个目录项，要遍历目录下的文件则使用 `while( (dirp = readdir(dp) != NULL) {...}` 循环。readdir 返回的是一个`dirent`结构指针，dirent 结构具体与实现有关，不过其中至少都定义了下面两个成员：

```
ino_t d_ino;
char d_name[];
```
d_name 项即指向一个目录项的文件名，它的大小没有指定，但必须保证它至少包含 NAME_MAX个字节。常用的读取目录示例：

```
if ((dp = opendir(fullpath)) == NULL) {
  ..
}

while ((dirp = readdir(dp)) != NULL) {
 	if (strcmp(dirp->d_name, ".") == 0 ||
         strcmp(dirp->d_name, "..") == 0)
    	continue;
 	...
}

if (closedir(dp) < 0) {
 printf("closr dir %s error: %s\n", fullpath, strerror(errno));
}
```

### 目录操作函数
修改当前(进程)工作目录函数：

```
#include <unistd.h>
int chdir(const char *pathname);
int fchdir(int fd);

char *getcwd(char *buf, size_t size);
```
这些函数只会影响进程本身，不会影响其它进程。 getcwd 返回当前工作目录的绝对路径。

---
彩蛋：一种 typedef 声明函数的方法

这是在APUE中讲目录遍历示例中使用的函数方法，非常的trick，平时不怎么能接触到，在这里分析一下，我修改了一些，其代码如下：

```
typedef int FileTypeFunc(const char *, const struct stat *, int);

static FileTypeFunc myfunc;

int main(void) {
	...
}

static int myfunc(const char *pathname, const struct stat *statptr, int type) {
	...
}
```
前面的文章提到了函数指针的用法，这里使用typedef方式也是类似的。

基本结束
