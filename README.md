# GDB 教程

快速入门gdb，如果要深入理解gdb，还是要从源码入手

## 内容

- [原理](#原理)
- [启动gdb](#启动gdb)
- [退出gdb](#退出gdb)
- [为gdb进行编译](#为gdb进行编译)
- [调试程序](#调试程序)
- [CoreDump简单概念](#CoreDump简单概念)
- [产生CoreDump文件](#产生CoreDump文件)
- [调试CoreDump文件](#调试CoreDump文件)
- [调试CoreDump文件](#调试CoreDump文件)
- [help命令](help命令)
- [list命令](list命令)
- [start命令](start命令)
- [run命令](run命令)


## 原理
断点功能一般是通过特定的内核信号实现的，gdb捕获该信号，定位目标程序停止的地址来判断断点是否成功触发。
首先gdb fork()出来一个子进程，该子启动目标程序(通过ptrace() 和 exec())，
父进程捕获该子进程的所有的信号(通过ptrace() 和 wait())，当子进程收到信号时，子进程就会被挂起，直到父进程通知其继续运行(通过ptrace())

## 启动gdb
1 常规启动，非常多的提示信息:

    $ gdb
    GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-114.el7
    Copyright (C) 2013 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "x86_64-redhat-linux-gnu".
    For bug reporting instructions, please see:
    <http://www.gnu.org/software/gdb/bugs/>.
    (gdb)
    
2 简约启动，关闭提示信息:

    $ gdb -q
    (gdb)

## 退出gdb
1 输入quit:

    $ gdb -q
    (gdb) quit

2 输入Ctrl-d:

    $ gdb -q
    (gdb) quit
    
    
## 为gdb进行编译

    为了获得调试信息，需要添加 CFLAGS=-g -o0 选项
具体参考[gdb手册](https://sourceware.org/gdb/current/onlinedocs/gdb/Compilation.html#Compilation)

## 调试程序

```c 
//boom.c
#include <stdio.h>
#include <unistd.h>

void fun(void)
{
    printf("hello\n");
}
    
int main()
{
    fun();
    sleep(1000);
    return 0;
}
```

1 直接启动:

方法1

    $ gdb boom -q
    (gdb) 

方法2

    $ gdb -q
    (gdb) file boom 
    Reading symbols from /home/dan/work/learn_core/build/bin/boom...done.
    (gdb)
   
2 调试正在运行的程序:
    
    $ ps ux | grep boom | grep -v 'grep' 
    dan       5647  0.0  0.0  11520   472 pts/0    S+   15:31   0:00 ./boom
    
    
    $ gdb boom 5647 -q
    Reading symbols from /home/dan/work/learn_core/build/bin/boom...done.
    Attaching to program: /home/dan/work/learn_core/build/bin/boom, process 5647
    Reading symbols from /lib64/libpthread.so.0...(no debugging symbols found)...done.
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib64/libthread_db.so.1".
    Loaded symbols for /lib64/libpthread.so.0
    Reading symbols from /lib64/libdl.so.2...(no debugging symbols found)...done.
    Loaded symbols for /lib64/libdl.so.2
    Reading symbols from /lib64/libm.so.6...(no debugging symbols found)...done.
    Loaded symbols for /lib64/libm.so.6
    Reading symbols from /lib64/libc.so.6...(no debugging symbols found)...done.
    Loaded symbols for /lib64/libc.so.6
    Reading symbols from /lib64/ld-linux-x86-64.so.2...(no debugging symbols found)...done.
    Loaded symbols for /lib64/ld-linux-x86-64.so.2
    0x00007f1e2185be10 in __nanosleep_nocancel () from /lib64/libc.so.6
    Missing separate debuginfos, use: debuginfo-install glibc-2.17-260.el7.x86_64
    (gdb) 

具体参考[gdb手册](https://sourceware.org/gdb/onlinedocs/gdb/Invoking-GDB.html#Invoking-GDB)

## CoreDump简单概念
CoreDump即核心转储，是程序运行异常崩溃时，系统内核为该程序产生的内存、寄存器、运行栈等快照，并保存为一个二进制文件，可以利用该文件进行GDB调试，发现运行错误。
    
## 产生CoreDump文件    
查看系统是否已经开启了该功能:
    
    $ ulimit -c
    0
上述输出结果为0说明当前系统已经关闭了该功能，所以需要打开该功能:

临时启用
    
    $ ulimit -c unlimited
    $ ulimit -c
    unlimited

永久启用

    在/etc/security/limits.conf添加一行:
    *       soft    core    unlimited
    
    修改文件格式，例如：
    echo "core.%e.%p.%t" >/proc/sys/kernel/core_pattern
    or
    echo "/home/dan/mycore/core.%e.%p.%t" >/proc/sys/kernel/core_pattern
    
## 调试CoreDump文件

```c 
#include <stdio.h>
#include <unistd.h>
 
int main()
{
    int* p = NULL;
    printf("here");
    *p = 7;
    sleep(1000);
    return 0;
}
 ```
运行上述程序就会产生core dump 文件，如果修了core dump文件的位置就需要加上绝对地址如:
    
方法1

    $ gdb /home/dan/work/learn_core/build/bin/boom /home/dan/mycore/core.boom.5859.1557305516 -q
    Reading symbols from /home/dan/work/learn_core/build/bin/boom...done.
    [New LWP 5859]
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib64/libthread_db.so.1".
    Core was generated by `./boom'.
    Program terminated with signal 11, Segmentation fault.
    #0  0x0000000000400770 in main () at /home/dan/work/learn_core/boom.c:8
    8           *p = 7;
    Missing separate debuginfos, use: debuginfo-install glibc-2.17-260.el7.x86_64
    (gdb) 

方法2

    gdb -q
    (gdb) file /home/dan/work/learn_core/build/bin/boom
    Reading symbols from /home/dan/work/learn_core/build/bin/boom...done.
    (gdb) core /home/dan/mycore/core.boom.5859.1557305516
    [New LWP 5859]
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib64/libthread_db.so.1".
    Core was generated by `./boom'.
    Program terminated with signal 11, Segmentation fault.
    #0  0x0000000000400770 in main () at /home/dan/work/learn_core/boom.c:8
    8           *p = 7;
    Missing separate debuginfos, use: debuginfo-install glibc-2.17-260.el7.x86_64
    (gdb) 
    
如果没有修改core dump文件的位置，该文件就会在程序的当前位置产生，就可以直接启动如:

方法1

    $ gdb boom core.boom.5941.1557306910 -q
    Reading symbols from /home/dan/work/learn_core/build/bin/boom...done.
    [New LWP 5941]
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib64/libthread_db.so.1".
    Core was generated by `./boom'.
    Program terminated with signal 11, Segmentation fault.
    #0  0x0000000000400770 in main () at /home/dan/work/learn_core/boom.c:8
    8           *p = 7;
    Missing separate debuginfos, use: debuginfo-install glibc-2.17-260.el7.x86_64
    (gdb) 
    
方法2

    $ gdb -q
    (gdb) file boom
    Reading symbols from /home/dan/work/learn_core/build/bin/boom...done.
    (gdb) core  core.boom.5941.1557306910
    [New LWP 5941]
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib64/libthread_db.so.1".
    Core was generated by `./boom'.
    Program terminated with signal 11, Segmentation fault.
    #0  0x0000000000400770 in main () at /home/dan/work/learn_core/boom.c:8
    8           *p = 7;
    Missing separate debuginfos, use: debuginfo-install glibc-2.17-260.el7.x86_64
    (gdb) 
    
## help命令
简写为h，查询命令帮助手册，例如:

    $ gdb -q
    (gdb) help start
    Run the debugged program until the beginning of the main procedure.
    You may specify arguments to give to your program, just as with the
    "run" command.
    (gdb) 

## start命令
start命令会给main函数的第一个可执行语句打上临时断点，然后运行程序直到该断点，例如:

```c
#include <stdio.h>
 
void func()
{
    printf("here");
}
      
int main()
{
    int a = 0;
    func();
    a++;
    return 0;
}
```
    
    gdb boom -q
    Reading symbols from /home/dan/work/learn_core/build/bin/boom...done.
    (gdb) start
    Temporary breakpoint 1 at 0x40074a: file /home/dan/work/learn_core/boom.c, line 10.
    Starting program: /home/dan/work/learn_core/build/bin/boom 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib64/libthread_db.so.1".

    Temporary breakpoint 1, main () at /home/dan/work/learn_core/boom.c:10
    10          int a = 0;
    Missing separate debuginfos, use: debuginfo-install glibc-2.17-260.el7.x86_64
    (gdb) 
    
# list命令
简写为l，查看源代码，例如:
```c
#include <stdio.h>
 
void func()
{
    printf("here");
}
      
int main()
{
    int a = 0;
    func();
    a++;
    return 0;
}
```
list num 指定行号

    $ gdb boom -q
    Reading symbols from /home/dan/work/learn_core/build/bin/boom...done.
    (gdb) l 7
    
    void func()
    {
        printf("here");
    }


    int main()
    {
        int a = 0;
 
 list function 指定函数名
 
    $ gdb boom -q
    Reading symbols from /home/dan/work/learn_core/build/bin/boom...done.
    (gdb) list func
    #include <stdio.h>

    void func()
    {
        printf("here");
    }

    int main()
    {
        int a = 0;
    (gdb) 
    
 list start,end 指定范围
 
    $ gdb boom -q
    Reading symbols from /home/dan/work/learn_core/build/bin/boom...done.
    (gdb) list 1,22
    #include <stdio.h>

    void func()
    {
        printf("here");
    }

    int main()
    {
        int a = 0;
        func();
        a++;
        return 0;
    }
    (gdb) 
 
 list + 向后打印
 list - 向前打印
