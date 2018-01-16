---
date: 2015/11/24 12:34:50
title: CSAPP：链接重定位
categories:
- programming
tags:
- csapp
- c
- linking
---
![](/images/post/csapp2ecover.jpg)

 前一篇讲了链接的基础，这里继续链接过程的重定位实现机制以及可执行目标文件的结构和后面的一些内容。

## 重定位
前面讲过了链接器的符号解析过程，在符号解析完成之后，代码中的每个符号引用和确定的一个符号定义联系起来了。此时，链接器就知道它的输入目标模块中的代码和数据节的确定大小。此时开始进行重定位，重定位将合并输入模块，并为每个符号分配运行时地址。重定位分两步进行：

1. 重定位节和符号定义。 链接器将所有相同类型的节进行合并，形成一个新的聚合节。例如将所有输入模块的 .data 节合并到一个节，这个节成为输出的可执行目标文件的 .data 节。然后链接器将新的存储器地址赋给新的聚合节。此时程序中每个指令和全局变量都有了一个运行时存储器地址了。
2. 重定位节中的符号引用。 对指令和全局变量（定义）重编码后，需要将输入模块的节（代码和数据）中的符号引用进行重定位，使它们指向第一步重定位后的运行时地址。这一步依赖于一个称为 relocation entry(重定位条目)的数据结构。这个条目由**汇编器**生成，链接器根据这个结构的内容确定符号引用的运行时地址。

### relocation entry
汇编器生产一个目标模块时，它并不知道数据和代码最终存放在存储器的什么位置，所以在汇编器遇到最终位置未知的目标引用时，就生成一个重定位条目（relocation entry），告诉链接器如何在合并成可执行文件时修改这个引用。代码的重定位条目保存在 .rel.text 中，数据的重定位条目存放在 .rel.data 中（见上一篇的可重定位目标文件结构）。

relocation entry的数据结构定义如下：

```c
typedef struct {
    int offset;
    int symbol:24,
        type:8;
} ELF32_Rel;
```
其中offset是需要被修改的引用的节偏移（相对于节的偏移），symbol是标识被修改的引用应该指向的符号（的地址），type是高职链接器如果修改新的引用。

type有11中，我们只需要知道最主要的两种方式，一种是 R_386_PC32，即使用一个32位PC（程序计数器）相对地址的引用。PC的值通常是存储器中下一条指令的地址。另一种方式R_386_32；重定位使用一个32位绝对地址的引用。

### 重定位符号引用
一种重定位算法的伪代码如下所示：

```c
foreach section s {
    foreach relocation entry r {
        refptr = s + r.offset;
        if (r.type == R_386_PC32) {
            refaddr = ADDR(s) + r.offset;
            *refptr = (unsigned) (ADDR(r.symbol) + *refptr - refaddr);
        }
        if (r.type = R_386_32)
            *refptr = (unsigned)(ADDR(r.symbol) + *refptr);
    }
}
```
其中ADDR(s)表示s的运行时地址（在节内部符号重定位时，符号和指令的运行时地址已经知道了）。ADDR(r.symbol)是符号的运行时地址。

**重定位PC相对引用** 对于PC相对位置的重定位。根据上面的代码分析：refptr是一个地址，其值 *refaddr 是相对定位的偏移的值（32位长度）。一定要注意第6行的计算， *refptr不要用refptr代入计算了，注意它是一个指针。汇编器会将call指令的引用的初始值设置为 -4 (0xfffffffc)（在不同指令大小和编码方式上可能不一样）。这是因为PC总是指向当前指令的下一条指令，指向 -4 允许链接器透明地重定位引用，而不用知道某一台机器的指令编码。这部分使用一个例子更加清晰，CSAPP Page 462的例子可以仔细看看。

**重定位绝对引用** 

## 可执行文件的结构和加载
链接器将多个目标模块合并成了一个可执行的二进制目标文件。这个二进制文件包含加载程序到存储器并运行它所需的所有信息。一个典型的ELF可执行文件结构如下图所示。

![](/images/post/exe-obj.png)

其结构整体和可重定位目标文件差不多，不过因为已经完成了重定位（称作完全链接的），所以没有了  .rel.data 和 .rel.text 节了。多了一个 .init 节，定义了一个_init函数，程序的初始化代码会调用它。

段头部表(segment header table)描述了可执行文件的连续的片(chunk)被映射到连续的存储器段的信息。

shell通过一个驻留在存储器中称为加载器(loader)的操作系统代码来运行生成的可执行文件，任何Unix程序都可以通过调用 `execve` 函数来调用加载器。

每个Unix程序都一个一个运行时存储器影响，在32位Linux系统中，代码段**总是**从地址 0x08048000 处（小端存储）开始。数据段是接下来的下一个4KB对齐的地址处，运行时堆在读/写段之后接下来的下一个4KB对其的地址处，并通过调用malloc库往上增长。用户栈总是总最大的合法用户地址开始，向下增长的，从栈的上部开始的段是操作系统驻留存储器的部分（内核）保留的。其表示如下图。

![](/images/post/linux-run-mem-img.png)

每个C程序中启动例程的伪代码如下：

```
0x080480c0 <_start>:
    call __libc_init_first
    call _init
    call atexit
    call main
    call _exit
```
加载工作的一个概述如下：Unix系统中的每个程序都运行在一个进程上下文中，有自己的虚拟地址空间。当shell运行一个程序时，（父）shell进程生成一个子进程，它是父进程的一个复制，子进程通过execve系统调用启动加载器。加载器删除子进程现有的虚拟存储器段，并创建一组新的代码、数据、堆和栈段。新的栈和堆段被初始化为0.通过将虚拟地址空间中的页映射到可执行文件的页大小的片，新的代码和数据段被初始化为可执行文件的内容。最后加载器跳转到 _start 地址，它最终会调用应用程序的 main 函数。除了一些头部信息，在加载过程中没有任何从磁盘到存储器的数据拷贝。知道CPU引用一个被映射的虚拟页才会进行拷贝，此时操作系统利用它的页面调度机制自动将页面从磁盘传送到存储器。

待续