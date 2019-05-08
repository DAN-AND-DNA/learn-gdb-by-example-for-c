# GDB 教程

快速入门gdb，如果要深入理解gdb，还是要从源码入手

## 内容

- [原理](#原理)
- [启动gdb](#启动gdb)
- [退出gdb](#退出gdb)
- [为gdb进行编译](#为gdb进行编译)
- [调试程序](#调试程序)
- [调试core文件](#调试core文件)


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
   
具体参考[gdb手册](https://sourceware.org/gdb/onlinedocs/gdb/Invoking-GDB.html#Invoking-GDB)
 

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
    
## 调试core文件


