---
title: "Linux 相关"
date: 2015-08-07T08:57:52+10:00
draft: true
categories: ["Computer Science"]
---

# Linux 相关

## 用户态和内核态

- 内核态：控制计算机的硬件资源，并提供上层应用程序运行的环境
- 用户态：上层应用程序的活动空间，应用程序的执行必须依托于内核提供的资源
- 系统调用：为了使上层应用能够访问到这些资源，内核为上层应用提供访问的接口

### 系统调用

- 系统调用是操作系统中的最小功能单位
- 系统调用与上层应用程序的关系
    - 需要通过多个系统调用才能完成一个上层应用
- 系统调用与公用函数库的关系
公用函数库实现对系统调用的封装，将简单的业务逻辑接口呈现给用户，方便用户调用，从这个角度上看，库函数就像是组成汉字的“偏旁”。
从特权级来区分内核态和用户态：
在CPU的所有指令中，有一些指令是非常危险的，如果错用，将导致整个系统崩溃。所以，CPU将指令分为特权指令和非特权指令，对于那些危险的指令，只允许操作系统及其相关模块使用，普通的应用程序只能使用那些不会造成灾难的指令。

intel cpu提供Ring0-Ring3四种级别的运行模式，Ring0级别最高，Ring3最低。Linux使用了Ring3级别运行用户态，Ring0作为 内核态。

用户态切换为内核态的三种情况：
系统调用
异常事件： 当CPU正在执行运行在用户态的程序时，突然发生某些预先不可知的异常事件，这个时候就会触发从当前用户态执行的进程转向内核态执行相关的异常事件，典型的如缺页异常。
外围设备的中断：当外围设备完成用户的请求操作后，会像CPU发出中断信号，此时，CPU就会暂停执行下一条即将要执行的指令，转而去执行中断信号对应的处理程序，如果先前执行的指令是在用户态下，则自然就发生从用户态到内核态的转换。
系统调用的本质其实也是中断，相对于外围设备的硬中断，这种中断称为软中断。从触发方式和效果上来看，这三种切换方式是完全一样的，都相当于是执行了一个中断响应的过程。但是从触发的对象来看，系统调用是进程主动请求切换的，而异常和硬中断则是被动的。


## 上下文切换

### 上下文切换的原因

- 当前线程的时间片用完
- 当前线程因为 IO 被阻塞
- 当前线程没有抢占到锁资源，被调度器挂起
- 用户代码挂起当前线程
- 硬件中断

### 减少上下文切换

1. 无锁并发编程
    - 多线程竞争会引起上下文切换，所以在使用多线程编程时，可以用一些办法来避免使用锁，比如将数据的 ID 按照 Hash 取模分段不同的线程处理不同段的数据
2. CAS算法。Java的Atomic包使用CAS算法来更新数据，而不需要加锁
3. 使用最少线程。避免创建不需要的线程，比如任务很少，但是创建了很多线程来处理，这样会造成大量线程都处于等待状态
4. 协程。在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换

## 命令行指令

* 注明“在 Linux/MacOS 下”的只在指定系统下生效，否则双平台都生效

## 进程

* top 显示系统中进程的资源占用情况

![top](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/Computer-Science/top.png)

* ldd 在 Linux 下查看程序使用了哪些共享链接库
* otool -L 在 MacOS 下查看程序使用了哪些共享链接库

![otool](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/Computer-Science/otool.png)

* strace 在 Linux 下查看程序运行时的系统调用
* dtruss 在 MacOS 下查看程序运行时的系统调用

* pstack 在 Linux 下跟踪每个进程的栈

## 内存

* free 在 Linux 下显示系统内存的使用情况
* memstat 在 Linux 下识别共享库的虚拟内存使用情况

## 硬盘

- iostat 闲适输入输出设备和 CPU 的使用情况
- du 显示目录和文件的大小
- df 显示文件系统的磁盘使用情况

## 文件

### 文件管理

* scp 远程文件拷贝
* rz Receive ZMODEM，在 Linux 下使用 ZMODEM 协议将文件批量上传到远程 Linux 服务器
* sz Send ZModem，在 Linux 下使用 ZMODEM 协议从远程 Linux 服务器批量下载文件

* cd 切换目录
* dirs 显示目录
* find 在指定目录下查找文件
* whereis 查找二进制文件

* read 从标准输入读取数据
* awk 处理文件的语言

### 文件编辑

* grep 使用正则表达式搜索文本

* cut 显示每行的第 m 到 n 个文字
* cat 输出文件
* more 按页显示文件
* less 按页显示文件，可以向前翻

## 网络

* netstat 命令用于显示各种网络相关信息

![netstat](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/Computer-Science/netstat.png)

* ifconfig 获取和修改网络接口配置信息

* iptables Linux 下的防火墙软件

