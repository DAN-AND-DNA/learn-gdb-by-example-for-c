# GDB 教程

快速入门gdb，如果要深入理解gdb，还是要从源码入手

## 内容

- [原理](#原理)
- [启动gdb](#启动gdb)
- [退出gdb](#退出gdb)
- [装载目标程序](#装载目标程序)

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
    
## 装载目标程序


