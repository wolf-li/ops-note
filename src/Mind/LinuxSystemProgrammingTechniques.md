# Linux 系统编程

## chapter1. estabilsh development envirement

- 开发OS: Linux 发行版
- tools:  git、gcc, make , gdb, valgrind
  - 安装
  apt install git build-essential gdb valgrind make -y
  dnf install git 'Development Tools' gdb valgrind make -y
  - 验证
  git -v
  gcc --version
  make --version
  gdb --version
  valgrind --version
- 字符处理函数 #include <stdlib.h>
  - atoi() 将字符串转换为整数
  - atol() 将字符串转换为长整形
  - atoll(). 将字符串转换为长长整形
  - atof() 将字符串转换为浮点型
- unix标准库函数 #include <unistd.h>
  - getopt() 函数返回它解析字符的实际字母
- man 手册
  - 手册
  1 命令
  2 system calls
  3 library 函数
  4 内核接口
  5 file format
  6 游戏
  7 Miscellaneous Information
  8 系统管理
  9 内核开发
  - 搜索 man 手册内容  arpopos
  - 简短了解某个函数可以使用 whatis

## chapter2 program script

- linux 执行程序返回值 使用 $? 获取 （0 执行成功， 其他执行失败）
- test 命令常见使用
  - 检测一个目录是否存在
    test -e /; echo $?
- C 程序退出方法
  - retrun
  - exit()
    一旦 exit() 被调用程序将以指定的值退出。如果在函数中调用了 exit() 该函数将不会返回到 main()
- 返回码
  - C 语言中 0 代表假（错误），其他任何值都视为真
- 常见 shell 退出码
  退出码   含义
  0       执行成功
  1       一般性错误
  2       shell 内建命令使用错误
  126     命令无法执行
  127     命令未发现
  128     无效的退出参数
  128+n   信号，128+信号
  130     程序捕捉到一个中断信号（128+2）
  137     程序捕捉到一个杀死信号（128+9）
  /usr/include/sysexit.h 文件未尾还列其他退出码
- 重定向标准输入、标准输出和标准错误
  - stdin 标准输入   <  或  0<  文件描述符 0 /dev/fd/0  /dev/stdio
  - stdout 标准输出 1> 或 > 文件描述符 1 /dev/fd/1  /dev/stdout
  - stderr 标准错误  2>  文件描述符 2 /dev/fd/2 /dev/stderr
  - /dev/null 黑洞（回收站）发送到该文件的所有内容都会消失
  - 例子
    wc < test  # wc 从 test 文件读取内容
    ls -l /tt  2> test  # 错误信息写入到 test
    ls -l / > log # 命令输出写入到 log 中
    ls -l / &> log1. # 标准输入和标准输出重定向到 log 文件中
- 使用管道连接程序
  shell 中的管道，管道连接的程序被称为过滤器
  - 例子：
  ls -l / | grep lrwx
- 写入标准输入和标准错误
  - 文件流：当我们想要写入文件时，在 C 程序中使用标准库打开的对象。
  - linux 中一切都是文件或进程，当一个程序在 Linux 中运行时，会自动打开三个文件流，stdin 、stdout、stderr
  - printf() 默认输出到标准输出中
  - fprintf() 指定文件流打印文本
  - dprintf() 指定文件描述符以打印输出信息 （需要使用 #define _POSIX_C_SOURCE 200809L）
  - Linux 中文件描述符和文件流
    名称     文件描述符号    文件流（stdio.h）    文件描述符（unistd.h）
    标准输入  0             stdin              STDIN_FILENO
    标准输出  1             stdout             STDOUT_FILENO
    标准错误  2             stderr             STDERR_FILENO
- 从标准输入读取
  char \*fgets (char \*str, int n, FILE \*stream)  从指定的流 stream 读取一行，并把它存储在 str 所指向的字符串内。
- 管道友好程序
  size_t strspn(const char *str1, const char \*str2) 检索字符串 str1 中第一个不在字符串 str2 中出现的字符下标。
- 结果重定向到文件
  (cat test.txt | awk '{print $3}' > | ./mph-to-kph-v2 -c) 2> error.txt 1> output.txt
- 读取环境变量
  C 库函数 char *getenv(const char \*name) 搜索 name 所指向的环境字符串，并返回相关的值给字符串。
  int setenv(const char \*name, const char \*value, int overwrite);

## chapter 3 linux c

- gcc 参数
  - gcc 使用链接库 -l.  使用数学库 math.h. -lm 这个在 linux 使用 gcc 需要
  - gcc -Wall -Wextra  -pedantic -fPIC -c prime.c 编译为目标文件对象 .o 结尾
  - gcc -shared -W1,-soname,libprime.so -o libprime.so prime.o 将目标文件打包成一个库
    - -shared 创建一个共享库
    - -W1,-soname,libprime.so 用于链接器，告诉链接器这个共享库的名字是 libprime.so
    - -o libprime.so ， -o 参数指定输出文件名，libprime.so 这是动态链接库的标准命名约定
  - -Wall 所有警告
  - -Wextra 额外警告
  - -pedantic 检查
  - -fPIC -f后面跟一些编译选项, PIC 表示生成位置无关代码
  - -std=99 使用通用的 C 标准如 C89 C99 C11
- 动态链接库标准命名
  - 按照共享库的命名惯例，每个共享库有三个文件名：real name、soname和linker name。
  - real name 真正共享库的名字不是符号链接。包含完整的共享库版本号。例如 libcap.so.1.10、libc-2.8.90.so等
  - soname 符号链接名字，只包含共享库的主版本号，主版本号一致即可保证库函数的接口一致，程序的 dynamic 段只记录共享库的 soname，只摇 soname 一致，这个共享库就可以使用。例如 libcap.so.1和libcap.so.2是两个主版本号不同的libcap。include /etc/ld.so.conf.d/*.conf 目录下指定linker name所在的目录。然后运行ldconfig命令，ldconfig会自动创建一个soname的符号链接
  - linker name 仅在编译链接时使用，gcc的-L选项应该指定linker name所在的目录。有的linker name是库文件的一个符号链接，有的linker name是一段链接脚本。
- 使用系统调用，本文所有系统调用指是指内核提供的 C 函数系统调用，而不是实际的系统调用表。使用的系统调用都是用户态调用的，但函数本身是在内核态执行的。 Linux 的系统调用都在 unistd.h 中声明。
  - ssize_t
     write(int fildes, const void *buf, size_t nbyte); fildes 文件描述符，buf 写入内容， nbyte 内容长度
- 获取 Linux 和 类Unix 头文件信息
  - apt install manpage-posix-dev
- 编译过程
  - 预处理 对文件内容进行修改
    gcc -E -P cube-func.c -o cube-func.i
  - 编译 预处理文件翻译成汇编语言
    gcc -S cube-prog.i -o cube-prog.s
  - 汇编 汇编代码构建为目标文件
    gcc -c cube-prog.s -o cube-prog.o
  - 链接 所有目标文件合并在一起
    gcc cube-prog.o cube-func.o -o cube
- make
  - make 可以找到源文件的名字   make 源文件 == cc 源文件.c -o 源文件
  - 简单 makefile
    CC=gcc
    CFLAGS=-Wall -Wextra -pedantic -std=c99

    cube: cube.h cube-prog.c cube-func.c
        $(CC) $(CFLAGS) -o cube cube-prog.c cube-func.c
  - 复杂 makefile
    CC=gcc
    CFLAGS=-Wall -Wextra -pedantic -std=c99
    LIBS=-lm  # 链接库
    OBJS=area.o help.o circle.o rectangle.o triangle.o  # 所有目标文件
    DEPS=area.h  
    bindir=/usr/local/bin

    area: $(OBJS)
        $(CC) -o area $(OBJS) $(LIBS)

    area.o: $(DEPS)

    clean:
        rm area $(OBJS)

    install: area
        install -g root -o root area $(bindir)/area

    uninstall: $(bindir)/area
        rm -f $(bindir)/area

## chapter 4 error handling

- errno
  在 Linux 和其他类 Unix 系统中，大多数系统调用函数在发生错误时会设置 errno 的特殊变量。通过这种方式通过返回值（通常是 -1）得到一个通用的错误代码通过查看 errno 变量获取错误的具体信息。
  需要使用 errno.h 头文件
  作用：保存特定的错误值
  错误宏有依赖性，需要部分函数支持才可以使用
  错误宏
  ECASS  没有权限
  ENOENT 路径不存在 (Error No Entry)
- strerror( int errno ) 将 errno 宏转换为可读错误消息减少代码
  if (create(filename, 00644) == -1){
    errornum = errno;
    fprintf(stderr, "Cant't create file %a\n", filename);
    fprintf(stderr, "%s\n", strerror(errornum));
    return 1;
  }
- perror(str)  将错误直接打印到错误输出 /dev/std2中, 直接使用即可不需要依赖 错误宏
- errno 小程序可以打印所有宏及其整数，默认情况下没有安装，安装包名称 moreutils
  - Debian 和 Ubuntu apt install moreutils
  - Fedora dnf install moreutils
  - CentOS 和 RedHat dnf install epel-release && dnf install moreutils

## chapter 5 IO

- 文件系统
  文件名不是实际文件，而是一个指向索引节点的指针。索引节点包含关于实际数据存储位置信息，文件的元数据，例如 文件模式，最后修改的日期、文件所有者。
- 查看文件的元数据  stat \<filename\>
- 读取索引节点信息
  使用 <sys/stat.h> 库 定义 stat 结构体和 stat函数获取 某个文件名的索引节点信息
  struct stat filestat;
  if (stat(argv[1], &filestat) == -1 ){
    fprintf(stderr, "Can't read file %s: \<file\>\n", argv[0]);
    return errno;
  }
  printf("Inode: %lu\n", filestat.st_ino);
  printf("Size: %zd\n", filestat.st_size);
  printf("Links: %lu\n", filestat.st_nlink);
- 创建软硬链接
  硬链接 文件名, 硬链接就是复制源文件的索引节点信息, 索引节点信息中的 link +1, ln \<source\> \<target\>
  软连接 文件名的快捷方式，与源文件索引节点信息一样, ln -s \<source\> \<target\>
  软连接创建利用系统函数 symlink()
  #include <unistd.h>
  int symlink(const char *target, const char \*linkpath);
  硬链接创建利用系统函数 link()
  #include <unistd.h>
  int link(const char \*oldpath, const char \*newpath);
- 创建文件并更新时间戳
  文件创建函数 creat() linux 可用
  #include <sys/types.h>
  #include <sys/stat.h>
  #include <fcntl.h>
  int creat(const char *pathname, mode_t mode);
- 更新文件时间戳利用系统函数 utime()
  #include <sys/types.h>
  #include <utime.h>
  int utime(const char *filename, const struct utimbuf \*times);
- 删除文件
  调用系统函数 unlink() 删除文件软硬链接都可以
  #include <unistd.h>
  int unlink(const char *pathname);
- 获得文件访问权限和所有权
  文件的完整文件模式由 6 个八进制数组成。前 2个 是文件类型，
  使用系统函数 stat() 获取文件访问权限和所有权 ，getpwuid() 函数通过 uid 获取用户信息，getgigid() 通过 gid 获取组信息 存储在<pwd.h> <grp.h> 中定义 passwd ， group 的结构体中
  文件类型
  s  14 socket
  l  12 symbolic link
  \-  10 regular file
  b  06 block device
  d  04 directory
  c  02 character derive
  p  01 FIFO 进程间通信使用
- 设置访问权限和所有权
  chmod() 系统调用设置访问权限
  #include \<sys/stat.h\>
  int chmod(const char *pathname, mode_t mode);
  chown() 系统调用更改文件所有者和组
  #include \<unistd.h\>
  int chown(const char \*pathname, uid_t owner, gid_t group);
- 使用文件描述符读取文件
  open 系统调用返回一个文件描述符
  #include <sys/types.h>
  #include <sys/stat.h>
  #include <fcntl.h>
  int open(const char \*pathname, int flags);
  int open(const char \*pathname, int flags, mode_t mode);
  write 向文件写入内容
  #include <unistd.h>
  ssize_t write(int fd, const void \*buf, size_t count);
  read 使用文件描述符读取文件内容
  #include <unistd.h>
  ssize_t read(int fd, void \*buf, size_t count);
- 使用流读写文件
  向流中写入文件
  #include <stdio.h>
  int printf(const char *format, ...);
  int fprintf(FILE \*stream, const char \*format, ...);
  从流中读取文件内容
- 使用流打开文件
  #include <stdio.h>
  FILE *fopen(const char \*pathname, const char \*mode);
  关闭流
  #include <stdio.h>
  int fclose(FILE \*stream);
- 使用流读写二进制文件
  #include <stdio.h>
  size_t fread(void \*ptr, size_t size, size_t nmemb, FILE \*stream);
  size_t fwrite(const void \*ptr, size_t size, size_t nmemb, FILE \*stream);
  使用 lseek() 在文件中移动
  #include <sys/types.h>
  #include <unistd.h>
  off_t lseek(int fd, off_t offset, int whence)
  使用 fseek() 在文件中移动
  #include <stdio.h>
  int fseek(FILE \*stream, long offset, int whence);

## chapter6 process

- 如何创建进程
Subtopic 1
	在 Bash 中使用作业控制
jobs 查看后台进程
fg  作业ID 进程前台显示
bg 作业ID  在后台继续运行该进程
	使用信号控制和终止进程
常用信号
	TERM
	KILL
	QUIT
	STOP
	HUP
	INT
	CONT
	在进程中使用execl() 替换运行程序
#include <unistd.h>
extern char \**environ;
int execl(const char \*pathname, const char \*arg, .../* (char  \*) NULL \*/);
	创建新进程
fork() 创建一个子进程
#include <sys/types.h>
#include <unistd.h>
pid_t fork(void);
	在创建进程中执行新进程
fork() 调用 execl() 
	使用 system() 启动一个新进程
#include <stdlib.h>
int system(const char \*command);
system 通过调用 sh 实现命令执行
	创建僵尸进程
僵尸进程：先于父进程退出的子进程，并且父进程没有等待子进程的状态。僵尸进程来自于进程没有死亡这个事实。进程已经退出，但在系统进程表中仍有该进程的条目
僵尸进程的危害：unix提供了一种机制可以保证只要父进程想知道子进程结束时的状态信息， 就可以得到。这种机制就是: 在每个进程退出的时候,内核释放该进程所有的资源,包括打开的文件,占用的内存等。 但是仍然为其保留一定的信息(包括进程号the process ID,退出状态the termination status of the process,运行时间the amount of CPU time taken by the process等)。直到父进程通过wait / waitpid来取时才释放。 但这样就导致了问题，如果进程不调用wait / waitpid的话， 那么保留的那段信息就不会释放，其进程号就会一直被占用，但是系统所能使用的进程号是有限的，如果大量的产生僵死进程，将因为没有可用的进程号而导致系统不能产生新的进程. 此即为僵尸进程的危害，应当避免。
解决方法：把产生大 量僵死进程的那个元凶枪毙掉（也就是通过kill发送SIGTERM或者SIGKILL信号啦）
	了解孤儿进程
孤儿进程：没有父进程的进程
每当出现一个孤儿进程的时候，内核就把孤 儿进程的父进程设置为systemd（旧版 init ），而systemd （旧版 init）进程会循环地wait()它的已经退出的子进程。这样，当一个孤儿进程凄凉地结束了其生命周期的时候，init进程就会代表党和政府出面处理它的一切善后工作。
	创建守护进程
守护进程； 在系统中后台执行的进程
	实现信号处理程序

## chapter 7 systemd

	手册
man systemd
man systemctl
man systemd.unit
man journalctl

## chapter 8 lib
	静态库
包含在二进制文件中，优点编译后，二进制文件独立与库，缺点文件大，库不能升级
静态库不能清除其符号，否则编译器不能找的函数链接失败
创建静态库
	gcc -Wall -Wextra -pedantic -std=c99 -c convert.c
	ar -cvr libconvert.a convert.o
系统添加静态库
	sudo install -o root -g root -m 644 libconvert.a /usr/local/lib/libconvert.a
编译使用静态库
	gcc -Wall -Wextra -pedantic -std=c99 -static temperature.c libconvert.a -o temperature-static.  // 可以使用 static 二进制包含库文件
	动态库
动态库会动态链接到使用它的二进制文件，库的代码不包含在二进制文件中，优点二进制文件小，库可以更新，不需要编译二进制程序。缺点移动库或删除库，二进制文件可能无法运行
系统添加动态库
	sudo install -o root -g root -m 755 libconvert.so.1 /usr/local/lib/libconvert.so.1
	sudo install -o root -g root -m 644 /home/li/convert.h /usr/local/include/convert.h
	sudo ldconfig.   创建一个库文件的符号链接
程序编译使用动态库
	gcc -Wall -Wextra -pedantic -std=c99 temperaturev2.c -o temperaturev2 -lconvert
查看程序使用的动态库
	$ ldd temperaturev2
	linux-vdso.so.1 (0x0000ffffb9823000)
	libconvert.so => /usr/local/lib/libconvert.so (0x0000ffffb97b0000)
	libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x0000ffffb9600000)
	/lib/ld-linux-aarch64.so.1 (0x0000ffffb97ea000)

## chapter 9 terminal I/O
	查看终端信息
tty // 查看正在使用系统的那个 TTY
file /dev/ttys002
/dev/ttys002: character special (16/2)
stty -a  // 查看终端的属性
	使用 stty 改变终端设置
stty -echo // 不返回回显
stty echo  // 显示回显
stty eof .  // eof 替换为 . 
	调查 TTY 和 PTY 设备，并向它们写入数据
echo "123" /dev/pts/2 // 大概会在终端中出现
使用root 用户
mesg y 允许或禁止消息
write  用户名  /dev/tty1
内容
^D
// 向这个用户的 tty1 终端发送消息
	检查是否是 TTY 设备
	创建一个 PTY 
PTY包含两部分，一个 是主设备（即伪终端设备 PTM）一个从设备 PTS
	关闭密码提示回显
	读取终端大小