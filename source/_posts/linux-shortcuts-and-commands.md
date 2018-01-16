---
date: 2015/12/12 12:34:50
title: Linux Shortcuts And Commands
categories:
- server
tags:
- linux
- Cli
- Newbie
---
![](/images/linux-logo.png)

来源： [http://www.unixguide.net/linux/linuxshortcuts.shtml](http://www.unixguide.net/linux/linuxshortcuts.shtml) 个人觉得非常好的一个Linux入门资料。待我慢慢翻译整理出来。

> 前面的部分很值得看看，后面的很多管理工具都已经过时了，看不看影响不大了。

## 介绍
This is a practical selection of the commands we use most often. Press <Tab> to see the listing of all available command (on your PATH). On my small home system, it says there are 2595 executables on my PATH.  Many of these "commands" can be accessed from your favourite GUI front-end (probably KDE or Gnome) by clicking on the right menu or button. They can all be run from the command line.  Programs that require GUI have to be run from a terminal opened under a GUI.

图例: 

- `<>` = 键盘上的单个特殊按键或者功能键. 比如用 <Ctrl> 表示 "control" 键. 
- *italic* = 斜体字用于文件名，或者需要你自己替换的变量. 
- fixed width = 等宽字体表示Linux命令和文件.

对Unix新手的提示: 
<!-- more -->
1. Linux是大小写敏感的，例如netscape 和 Netscape 是两个不同的命令。文件名也是区分大小写的（Windows不区分文件名的大小写）。登录的用户名和密码也是区分大小写的（大小写敏感是Unix系统和C语言的传统） 
2. 文件名最多可以有256个字符，允许字符，数字，.(dot), _(underscore), -(dash)和其他的一些字符，（实际上Linux文件名可以用除了/之外的所有字符）. 
3. 以 .(dot0) 开始的文件（或者目录）默认不会在 `ls` 命令 的输出中显示，可以认为这些是隐藏文件。使用 `ls -a` (list with the option all)显示这些文件。
4. `/` 和 DOS系统中的 `\` （当然Windows NT中没有这个了）是等价的，意味着根目录。 
5. 在Linux中，所有目录都出现在一个单独的目录树，而没有Windows风格的盘符。
6. 在配置文件中，以 `#` 开头的行表示注释。（有的应用的配置文件可以使用 `;` 开头表示注释）

## 7.1 Linux 必备快捷键和常用命令（sanity commands)

    <Ctrl><Alt><F1> 
切换到第一个文本中断，在Linux系统下，默认能同时打开了有6个终端（这只在本地终端有效，SSH连接无效）

	<Ctrl><Alt><Fn> (n=1..6) 
切换到第n个文本终端

	tty 
输出当前在使用的终端的名字

	<Ctrl><Alt><F7> 
如果安装了X服务，可以切换到GUI（图形用户界面）

	<Ctrl><Alt><Fn> (n=7..12) 
切换到其他的GUI终端，默认在8-12没有运行GUI，但是可以在这里启动一个新的GUI终端。

	<Tab> 
在文本终端下（命令行）下的自动补全命令快捷键，如果后面只有一个可选命令则自动填写，如果有多个则列出所有可选命令。**This Shortcut IS GREAT**(如果看到一个Linux用户不会使用tab补全，尽情地嘲笑他吧)

	<ArrowUp> 
历史命令回滚，每按一次向上箭头，回到上一条命令并可以编辑它，回车执行这条命令。（这在gitbash中也可用，非常常用的操作）

	<Shift><PgUp> 
在一些老的终端中（非伪终端），比如老的ubuntu  server,无图形界面Linux Server中，输出没有滚动条，要想查看已经滚出去的内容，需要使用Shift+PageUp 的组合键

	<Shift><PgDown> 
终端输出下滚一屏

	<Ctrl><Alt><+>
在 X-window 系统系统中，切换到下一个 X-server分辨率方案（the next X-server resolution）（如果设置了多个的话）


> For multiple resolutions on my standard SVGA card/monitor, I have the following line in the file /etc/X11/XF86Config (the first resolution starts on default, the largest determines the size of the "virtual screen"): Modes "1024x768" "800x600" "640x480" "512x384" "480x300" "400x300" "1152x864"
	
	<Ctrl><Alt><-> 
在X-window中切换到上一个分辨率方案

	<Ctrl><Alt><BkSpc> 
在X-window中终结当前的X-window服务，当桌面环境奔溃的时候使用。

	<Ctrl><Alt><Del> 
关闭系统并重启

	<Ctrl>c 
终结（kill)当前执行的进程

	<Ctrl>d 
从当前终端退出，对于通过SSH连接的终端，Ctrl+d会退出登录。**非常非常非常好用的快捷键**

	<Ctrl>d 
在进程执行等待（比如等待输入）时，按下 Ctrl+d 发送 End-of-File 信号给进程。

	<Ctrl>s 
停止传输到终端，类似锁屏，Vim的初学者容易使用 Ctrl+s 保存文件导致“锁屏”，对输入没有反应。

	<Ctrl>q 
继续（恢复）到终端的传输，如果你的终端对输入没有反应了，试试这个组合键

	<Ctrl>z 
将当前执行进程放到后台执行

	exit 
退出当前登录，

	reset 
重置终端，这个以前很少用， 常用 Ctrl+l 清空屏幕，但是对于SSH登录仍可以使用滚轮会看历史输出，使用reset指令则会是一个全新的终端，丢掉了历史输出。

	<MiddleMouseButton> 
鼠标中键用于粘贴剪切板的内容。这个在不同的伪终端和SSH工具中可能设置不同。Paste the text which is currently highlighted somewhere else. This is the normal "copy-paste" operation in Linux.   Best used with a Linux-ready 3-button mouse or else set "3-mouse button emulation").

	~ 
波浪号表示当前用户的家目录，直接输入波浪号回车就能设置工作目录为家目录，或者 `cd` 直接回车也是同样的效果。

	. 
点号表示当前目录，比如 `./a.out` 表示运行当前路径下的 a.out 文件。注意一般默认没有把 . 目录添加到 path 环境变量中，所以执行当前路径下的可执行文件一般需要加上 `./` .

	.. 
两个点号表示当前目录的设计目录， `cd ..` 会跳转到上级目录，注意跟目录也有一个 .. 项（通过 `ls -a /` 查看），但是根目录的 ..  条目指向的仍然是根。

## 7.2 常用命令：系统信息

	pwd 
这是Print working directory 的首字母缩写，不要理解为password。表示打印出当前路径

	hostname
打印本地主机名，现在也可以用它设置主机名了。旧的终端可能使用`netconf`命令这种主机名

	whoami
打印当前用户的登录名

	id username 
id命令，输出 username 的uid，有效id 和 gid 和追加（附件）组信息。如果不带username参数，默认打印当前用户的信息

	date
打印当前系统时间，或者也可以用于设置时间 ，比如使用命令 `date 123123572000` 设置系统时间为 2000-12-31 23:57。在Linux可以使用 `setclock` 命令设置硬件（BIOS）的时间，不过现在的系统好像没有这个命令了。

	time 
显示当前终端的执行的CPU时间，用户时间等信息，注意容易与date命令混淆，因为常会以为time指令是输出当前系时钟`time ls`会显示执行ls这条命令的时间开销。

	who 
显示当前登录到这台机器的用户

	rwho -a 
现在好像没有这条命令了，用于返回所有登录的网络用户（all users logged on your network）

	finger user_name 
输出用户相关的信息

	last 
输出登录到系统的用户的记录（最近登录在前），使用 `-n 10` 参数限制输出的条数

	history | more 
输出当前用户指令执行的历史记录，（其具体行为与使用bash实现有关）

	uptime 
上一次启动后系统运行的时间

ps 
这是 print status的缩写，可不是Photoshop。列出当前用户执行的进程，常用参数 `-aux`。结合ps和grep指令用来查找某个进程的pid

	ps axu | more 
列出当前所有在执行的进程

	top 
一个最基础的任务管理器，大部分的Linux发行版默认安装top，当然还有很多更好用的top工具可以使用，比如 htop, atop，都可以自己安装，htop比较好用

	uname -a 
uname是Unix name的意思，-a 表示所有，输出系统的内核信息。对于有x-window的终端，可以使用guname获得更多信息

	free 
内存使用统计，现在可以使用 `-h` 参数，使输出更humanity。

	df -h 
df是disk free的简写，输出所有文件系统的磁盘信息（分区挂载信息等）。-h 也是 huma-readable的意思，否则输出磁盘的块区数据，看不懂

	du / -bh | more 
du是disk usage的首字母。输出磁盘使用情况，常用 `-sh`参数，-s表示summary只输出路径的文件大小总和而不一一列出所有的文件。

	cat /proc/cpuinfo 
/proc 目录下是一些系统运行时相关的信息，我们知道这个目录并不真实存在于磁盘上，是运行时系统，运行在内存之中。里面的文件是用户为访问内核信息提供了一个可读的借口。显然这个命令输出CPU相关的信息

	cat /proc/interrupts 
类似上一条，列出在用的中断列表，对于多核CPU可以看出各个和处理各类中断的统计情况，可以依次进行一些编程的优化（在Linux就是这个范中提过，不过一般人不需要掌握）

	cat /proc/version 
Linux 版本和其他信息，和uname的输出很相似，不过信息更多

	cat /proc/filesystems 
输出当前使用文件系统信息，不常用

	cat /etc/printcap 
配置的打印机信息，不常用

	lsmod 
加载的内核模块信息	

	set|more 
set打印所有的用户环境配置，内容超级多


	echo $PATH 
echo $variable_name 打印环境变量配置的一个变量的值。使用set可以设置新的环境变量，echo输出，这里echo $PATH 即是输出配置的系统路径

	dmesg | less 
dmesg是系统启动时dump下来的记录信息。Print kernel messages (the content of the so-called kernel ring buffer). Press "q" to quit "less". Use less /var/log/dmesg  to see what "dmesg" dumped into this file right after the last system bootup. 
 

## 7.3 基本操作

	any_command --help |more 
通用的获取命令帮助信息的方法，有的命令可能使用 -h 即可，这是两种不同风格的命令行参数方式，--help更被广泛使用，也可以使用`man`查看用户手册

	apropos *topic* 
列出所有个与topic有关的命令，`apropos ls`列出所有与ls有关的命令。常是命令说明中有相关字符的命令。

	help command 
显示一个内置命令的简要信息，有的终端程序没有这个命令了（zsh没有）

	ls 
ls是list的意思，最常用的命令，常用参数 `-alhtr`,分别代表：all, long form, human-readable, sort by time,reverse(逆序)。通常为ls设置一个别名为 `ls --color`

	ls -al |more 
见上

	cd directory 
cd即Change directory. 用于切换到指定路径。

	cp source destination 
cp即时Copy，用于复制文件。复制一个文件夹(路径下所有文件）需要使用 `-r` 参数表示 recursive（递归复制）。

	mcopy source destination 
这个不常用。Copy a file from/to a DOS filesystem (no mounting necessary). E.g., mcopy a:\autoexec.bat ~/junk . See man mtools for related commands: mdir, mcd, mren, mmove, mdel, mmd, mrd, mformat ....

	mv source destination 
mv就是Move。用于移动或者重命名一个文件或目录，对于目录操作不需要 -r。

	ln source destination 
ln是link的意思。用于创建连接。连接有点类似于Windows系统的快捷方式，但是他们是完全不一样的。连接包括软连接和硬链接。这里内容比较多了。看下面的引用吧.

> Create a hard link called destination to the file called source. The link appears as a copy of the original files, but in reality only one copy of the file is kept, just two (or more) directory entries point to it. Any changes the file are automatically visible throughout. When one directory entry is removed, the other(s) stay(s) intact. The limitation of the hard links are: the files have to be on the same filesystem, hard links to directories or special files are impossible.

	ln -s source destination 
创建一个软连接,软连接是非常有用的，在配置文件，路径配置，共享文件配置都很常用。

> Create a symbolic (soft) link called "destination" to the file called "source". The symbolic link just specifies a path where to look for the file. In contradistinction to hard links, the source and destination don't not have to tbe on the same filesystem. In comparison to hard links, the drawback of symbolic links are: if the original file is removed, the link is "broken", symbolic links can also create circular references (like circular references in spreadsheets or databases, e.g., "a" points to "b" and "b" points back to "a").

	rm files 
rm即Remove，用于删除文件. 常用参数 `-rf`， r表示递归删除，用于删除文件夹，f表示force，强制删除，谨慎同时使用这两个参数，因为会直接删除文件而没有提示，删错了就只能库了，尤其是 `rm -rf /` 这样的操作将会是致命的，所以一般会为将rm设置为`rm -i`的别名，但是这样有时候又太麻烦了。

	mkdir directory 
Make a new directory.用于创建新的文件夹，目录。可以使用 `-p`参数创建多层目录。

	rmdir directory 
Remove an empty directory.

	rm -r files 
递归删除文件，目录及其子目录。 Linux目前来说并没有回收站这种东西，删掉了基本就找不回来了，所以rm之前一定要好好考虑清楚，**谨慎操作**。

	cat filename | more 
将一个文本本件的内容打印出来。使用more一次只打印一份，不过对于现代的伪终端可以使用鼠标滚轮了，一般more用的比较少了。


	less filename 
Scroll through a content of a text file. Press q when done. "Less" is roughly equivalent to "more" , the command you know from DOS, although very often "less" is more convenient than "more".

	find / -name "filename" 
查找文件。Find the file called "filename" on your filesystem starting the search from the root directory "/". The "filename" may contain wildcards (*,?).

	locate filename 
更方便的全局文件查找，维护了一个数据库，但是可能要手动更新这个数据库（使用`updatedb`命令）数据库默认在夜间自动更新。现在新的内核已经不再默认安装这个命令。

	./program_name 
Run an executable in the current directory, which is not on your PATH.

	touch filename 
如果文件已经存在则修改文件的时间戳到当前系统时间，如果不存在该文件则新建一个该文件。

	startx 
启动一个x-windows server。

	startx -- :1 
Start another X-windows session on the display 1 (the default is opened on display 0). You can have several GUI terminals running concurrently. Switch between them using <Ctrl><Alt><F7>, <Ctrl><Alt><F8>, etc.

	xterm 
(in X terminal) Run a simple X-windows terminal.  Typing exit will close it.  There are other, more advanced "virtual" terminals for X-windows. I like the popular ones: konsole and kvt (both come with kde) and gnome-terminal (comes with gnome).  If you need something really fancy-looking, try Eterm.

	shutdown -h now 
用root身份运行，关闭系统。

	halt 
	reboot 
(as root, two commands) Halt or reboot the machine. Used for remote shutdown, simpler to type than the previous command. 

## 网络应用（有删减，很多工具不存在或者已经不用了）


	lynx file.html 
lynx是一个基于文本的浏览器。用来查看html文件，默认没有安装。

	ftp server 
ftp登录到服务器。

> Ftp another machine. (There is also ncftp which adds extra features and gftp for GUI .) Ftp is good for copying files to/from a remote machine. Try user "anonymous" if you don't have an account on the remote server. After connection, use "?" to see the list of available ftp commands.  The essential ftp command are: ls (see the files on the remote system), ASCII, binary (set the file transfer mode to either text or binary, important that you select the proper one ), get (copy a file from the remote system to the local system), mget (get many files at once), put (copy a file from the local system to the remote system), mput (put many files at once), bye (disconnect). For automation in a script, you may want to use ncftpput and ncftpget, for example: 
ncftpput -u my_user_name -p my_password -a remote.host.domain remote_dir *local.html

### 文件压缩与解压

	tar -zxvf filename.tar.gz 
解压一个gunzip压缩的文件。

	tar -xvf filename.tar 
untar一个**没有压缩**的打包文件。

	gunzip filename.gz 
使用unzip解压一个压缩文件。

	bunzip2 filename.bz2 
bzip即big zip。解压一个使用bzip2方法压缩的文件，现在应该可以用 `untar -zjvf ****`解压这类文件了。

	unzip filename.zip 
Decompress a file (*.zip) zipped with a compression utility compatible with PKZIP for DOS.

	unarj e filename.arj 
很少用了。Extract the content of an *.arj archive.

uudecode -o outputfile filename 
没用过。Decode a file encoded with uuencode.  uu-encoded files are typically used for transfer of non-text files in e-mail (uuencode transforms any file into an ASCII file).

## 进程控制

	ps 
这在前面已经介绍过了。

	fg PID 
将一个后台进程调度到前台（终端可见）

	bg PID 
将一个进程放到后台去。也可以使用 Ctrl+z实现。

	any_command& 
在命令后加一个 `&`让命令后台执行。不过命令的输出信息还是会在终端打印出来。

	batch any_command 
Run any command (usually one that is going to take more time) when the system load is low. I can logout, and the process will keep running.

	at 17:00 
Execute a command at a specified time.  You will be prompted for the command(s) to run, until you press <Ctrl>d.

	kill PID 
Force a process shutdown. First determine the PID of the process to kill using ps.

	killall program_name 
Kill program(s) by name.

nice program_name 
有意思的一个指令，设置指令的优先级。

> Run program_name adjusting its priority. Since the priority is not specified in this example, it will be adjusted by 10 (the process will run slower), from the default value (usually 0). The lower the number (of "niceness" to other users on the system), the higher the priority. The priority value may be in the range -20 to 19. Only root may specify negative values. Use "top" to display the priorities of the running processes.

	renice -1 PID 

> (as root) Change the priority of a running process to -1. Normal users can only adjust processes they own, and only up from the current value (make them run slower).

<Ctrl>c, <Ctrl>z, <Ctrl>s, and <Ctrl>q also belong to this chapter but they were described previously. In short they mean: stop the current command, send the current command to the background, stop the data transfer, resume the data transfer. 
 

## 7.5 基本的管理命令

	printtool 
(as root in X-terminal) Configuration tool for your printer(s). Settings go to the file /etc/printcap.

	setup
系统配置命令，图形界面的。setup is the default on RedHat. Mandrake 7.0 offers very nice DrakConf .

	alias ls="ls --color=tty" 
创建一个别名，这个很常用，常将别名写在 `/etc/profile` 或者 `~/.bashrc` 中使得每次都生效。

	adduser user_name 
创建用户。创建用户后再使用`passwd`命令添加密码。会自动创建家目录。

	useradd user_name 
The same as the command " adduser user_name ".

	userdel user_name 
删除用户。用户家目录和邮件目录等需要另外删除（不会自动删除）

	groupadd group_name 
添加用户组。

	passwd [username]
修改用户密码，不带用户名做参数则修改当前用户密码。

	chmod perm filename 
chmod即change mode，修改文件的访问权限。

>  Change the file access permission for the files you own (unless you are root in which case you can change any file). You can make a file accessible in three modes: read (r), write (w), execute (x) to three classes of users: owner (u), members of the same group as the owner (g), others on the system (o). Check the current access permissions using: 
> ls -l filename 
> If the file is accessible to all users in all modes it will show: 
> rwxrwxrwx 
> The first triplet shows the file permission for the owner of the file, the second for his/her group, the third for others. A "no" permission is shown as "-". 
> E.g., this command will add the permission to read the file "junk" to all (=user+group+others): 
> chmod a+r junk 
> This command will remove the permission to execute the file junk from others: 
> chmod o-x junk 
> Also try here for more info. 
> You can set the default file permissions for the news files that you create using the command umask (see man umask).

	chown new_ownername filename 
	chgrp new_groupname filename 
修改文件的所属用户与组。通常在复制文件后用来修改权限。

	su 
切换到root用户

> (=substitute user id) Assume the superuser (=root) identity (you will be prompted for the password). Type "exit" to return you to your previous login. Don't habitually work on your machine as root. The root account is for administration and the su command is to ease your access to the administration account when you require it. You can also use "su" to assume any other user identity, e.g. su barbara will make me "barbara" (password required unless I am a superuser).

lsmod 
前面已经介绍过了。List currently loaded kernel modules. A module is like a device driver--it provides operating system kernel support for a particular piece of hardware or feature.

	modprobe -l |more 
列出所有可用的内核模块。List all the modules available for your kernel. The available modules are determined by how your Linux kernel was compliled. Every possible module/feature can be compiled on linux as either "hard wired" (fast, non-removable), "module" (maybe slower, but loaded/removable on demand), or "no" (no support for this feature at all).

	insmod parport 
	insmod ppa 
安装内核模块。

	rmmod module_name 
卸载内/移除内核模块

	fdisk 
磁盘分区工具。

	ldconfig 
(as root) Re-create the bindings and the cache for the loader of dynamic libraries ("ld"). You may want to run ldconfig after an installation of new dynamically linked libraries on your system. (It is also re-run every time you boot the computer, so if you reboot you don't have to run it manually.)

	mknod /dev/fd0 b 2 0 
(=make node, as root) Create a device file. This example shows how to create a device file associated with your first floppy drive and could be useful if you happened to accidentally erase it. The options are: b=block mode device (c=character mode device, p=FIFO device, u=unbuffered character mode device). The two integers specify the major and the minor device number.

	fdformat /dev/fd0H1440 
	mkfs -c -t ext2 
(=floppy disk format, two commands, as root) Perform a low-level formatting of a floppy in the first floppy drive (/dev/fd0), high density (1440 kB). Then make a Linux filesystem (-t ext2), checking/marking bad blocks (-c ). Making the files system is an equivalent to the high-level format.

	badblocks /dev/fd01440 1440 
(as root) Check a high-density floppy for bad blocks and display the results on the screen. The parameter "1440" specifies that 1440 blocks are to be checked. This command does not modify the floppy.

	fsck -t ext2 /dev/hda2 
(=file system check, as root) Check and repair a filesystem. The example uses the partition hda2, filesystem type ext2.

	dd if=/dev/fd0H1440 of=floppy_image 
	dd if=floppy_image of=/dev/fd0H1440 
dd="data duplicator"。Create an image of a floppy to the file called "floppy_image" in the current directory. Then copy floppy_image (file) to another floppy disk. Works like DOS "DISKCOPY". 
 
### 程序安装

	rpm -ivh filename.rpm 
安装指定的rpm包。(=RedhatPackageManager, install, verbose, hashes displayed to show progress, as root.) Install a content of RedHat rpm package(s) and print info on what happened. Keep reading if you prefer a GUI installation.

	rpm -qpi filename.rpm 
查询包。(=RedhatPackageManager, query, package, list.) Read the info on the content of a yet uninstalled package filename.rpm.

	rpm -qpl filename.rpm 
(=RedhatPackageManager, query, package, information.) List the files contained in a yet uninstalled package filename.rpm.

	rpm -qf filename 
(=RedhatPackageManager, query, file.) Find out the name of the *.rpm package to which the file filename (on your hardrive) belongs.

	rpm -e packagename 
卸载rpm包(=RedhatPackageManager, erase=uninstall.) Uninstall a package pagckagename. Packagname is the same as the beginning of the *.rpm package file but without the dash and version number.

### Accessing drives/partitions

	mount 
See here for details on mounting drives.  Examples are shown in the next commands.
mount -t auto /dev/fd0 /mnt/floppy 
(as root) Mount the floppy. The directory /mnt/floppy must exist, be empty and NOT be your current directory.

	mount -t auto /dev/cdrom /mnt/cdrom 
(as root) Mount the CD. You may need to create/modify the /dev/cdrom file depending where your CDROM is. The directory /mnt/cdrom must exist, be empty and NOT be your current directory.

	mount /mnt/floppy 
(as user or root) Mount a floppy as user. The file /etc/fstab must be set up to do this. The directory /mnt/floppy must not be your current directory.

	mount /mnt/cdrom 
(as user or root) Mount a CD as user. The file /etc/fstab must be set up to do this. The directory /mnt/cdrom must not be your current directory.

	umount /mnt/floppy 
Unmount the floppy. The directory /mnt/floppy must not be your (or anybody else's) current working directory. Depending on your setup, you might not be able to unmount a drive that you didn't mount. 
 

## 7.6 网络管理工具（大改）

**这里的大部分工具都过时了，就不整理了，新的网络管理工具参考之前的一篇[介绍ss和ip的文章](http://wuxu92.github.io/linux-ip-command)**

	netconf 
(as root) A very good menu-driven setup of your network.
pingmachine_name 
Check if you can contact another machine (give the machine's name or IP), press <Ctrl>C when done (it keeps going).

	route -n 
Show the kernel routing table.

	nslookup host_to_find 
Query your default domain name server (DNS) for an Internet name (or IP number) host_to_find. This way you can check if your DNS works. You can also find out the name of the host of which you only know the IP number.

	traceroute host_to_trace 
Have a look how you messages trave to host_to_trace (which is either a host name or IP number).

	ipfwadm -F -p m 
(for RH5.2, seen next command for RH6.0) Set up the firewall IP forwarding policy to masquerading. (Not very secure but simple.) Purpose: all computers from your home network will appear to the outside world as one very busy machine and, for example, you will be allowed to browse the Internet from all computers at once.

	echo 1 > /proc/sys/net/ipv4/ip_forward 
ipfwadm-wrapper -F -p deny 
ipfwadm-wrapper -F -a m -S xxx.xxx.xxx.0/24 -D 0.0.0.0/0 
(three commands, RH6.0). Does the same as the previous command. Substitute  the "x"s  with digits of your class "C" IP address that you assigned to your home network. See here for more details. In RH6.1, masquarading seems broken to me--I think I will install Mandrake Linux:).

	ifconfig 
(as root) Display info on the network interfaces currently active (ethernet, ppp, etc). Your first ethernet should show up as eth0, second as eth1, etc, first ppp over modem as ppp0, second as ppp1, etc. The "lo" is the "loopback only" interface which should be always active. Use the options (see ifconfig --help) to configure the interfaces.

	ifup interface_name 
(/sbin/ifup to it run as a user) Startup a network interface. E.g.: 
ifup eth0 
ifup ppp0 
Users can start up or shutdown the ppp interface only when the right permission was checked during the ppp setup (using netconf ). To start a ppp interface (dial-up connection), I normally use kppp available under kde menu "internet".

	ifdown interface_name 
(/sbin/ifdown to run it as a user). Shut down the network interface. E.g.: ifdown ppp0 Also, see the previous command.

	netstat | more 
Displays a lot (too much?) information on the status of your network. 
 

Music-related commands

	cdplay play 1 
Play the first track from a audio CD.

	eject 
Get a free coffee  cup holder :))).   (Eject the CD ROM tray).

	play my_file.wav 
Play a wave file.

	mpg123 my_file.mp3 
Play an mp3 file.

	mpg123 -w my_file.wav my_file.mp3 
Create a wave audio file from an mp3 audio file.

	knapster 
(in X terminal) Start the program to downolad mp3 files that other users of napster have displayed for downloading. Really cool!

	cdparanoia -B  "1-" 
(CD ripper)  Read the contents of an audio CD and save it into wavefiles in the current directories, one track per wavefile.  The "1-" 
means "from track 1 to the last". -B forces putting each track into a separate file.

	playmidi my_file.mid 
Play a midi file.  playmidi -r my_file.mid  will display text mode effects on the screen.

	sox 
(argument not given here) Convert from almost any audio file format to another (but not mp3s).  See man sox. 
 

Graphics-related commands

	kghostview my_file.ps 
Display a postscript file on screen.  I can also use the older-looking ghostview or gv for the same end effect.
ps2pdf my_file.ps my_file.pdf 
Make a pdf (Adobe portable document format) file from a postscript file.

	gimp 
(in X terminal) A humble looking but very powerful image processor. Takes some learning to use, but it is great for artists, there is almost nothing you can't do with gimp. Use your mouse right button to get local menus, and learn how to use layers. Save your file in the native gimp file format *.xcf (to preserve layers) and only then flatten it and save as png (or whatever).  There is a large user manual /usr/

	gphoto 
(in X terminal) Powerful photo editor.

giftopnm my_file.giff > my_file.pnm 
pnmtopng my_file.pnm > my_file.png 
Convert the propriatory giff graphics into a raw, portable pnm file. Then convert the pnm into a png file, which is a newer and better standard for Internet pictures  (better technically plus there is no danger of being sued by the owner of giff patents).

后面的很多东西都比较过时了，so 没有整理了。前面的部分还是很值得看的。
