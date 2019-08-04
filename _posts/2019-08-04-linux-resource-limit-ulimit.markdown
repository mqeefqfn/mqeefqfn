---
layout: post
title:  "(转)linux最大进程数、最大线程数、进程打开的文件数和ulimit命令"
date:   2015-11-14 16:52:07
categories: linux
tags: ulimit 
# image: /images/pic03.jpg
---

# 1. ulimit命令查看和更改系统限制

ulimit用于shell启动进程所占用的资源，可以用来设置系统的限制

语法格式

> ulimit [-acdfHlmnpsStvw] [size]

在`/etc/security/limits.conf`文件中定义限制。



| 命令参数 |描述 | 例子 |
| ---- | ---- | ---- |
|-H|	设置硬资源限制，一旦设置不能增加.| ulimit – Hs 64；限制硬资源，线程栈大小为 64K。|
|-S|设置软资源限制，设置后可以增加，但是不能超过硬资源设置。|ulimit – Sn 32；限制软资源，32 个文件描述符。|
|-a|显示当前所有的 limit 信息|ulimit – a；显示当前所有的 limit 信息|
|-c	|最大的 core 文件的大小， 以 blocks 为单位|ulimit – c unlimited； 对生成的 core 文件的大小不进行限制|
|-d	|进程最大的数据段的大小，以 Kbytes 为单位 |ulimit -d unlimited；对进程的数据段大小不进行限制|
|-f	|进程可以创建文件的最大值，以 blocks 为单位	|ulimit – f 2048；限制进程可以创建的最大文件大小为 2048 blocks|
|-l	|最大可加锁内存大小，以 Kbytes 为单位| ulimit – l 32；限制最大可加锁内存大小为 32 Kbytes|
|-m	|最大内存大小，以 Kbytes 为单位|ulimit – m unlimited；对最大内存不进行限制|
|-n	|可以打开最大文件描述符的数量| ulimit – n 128；限制最大可以使用 128 个文件描述符|
|-p	|管道缓冲区的大小，以 Kbytes 为单位| ulimit – p 512；限制管道缓冲区的大小为 512 Kbytes|
|-s	|线程栈大小，以 Kbytes 为单位|ulimit – s 512；限制线程栈的大小为 512 Kbytes|
|-t	|最大的 CPU 占用时间，以秒为单位| ulimit – t unlimited；对最大的 CPU 占用时间不进行限制|
|-u	|用户最大可用的进程数|ulimit – u 64；限制用户最多可以使用 64 个进程|
|-v	|进程最大可用的虚拟内存，以 Kbytes 为单位| ulimit – v 200000；限制最大可用的虚拟内存为 200000 Kbytes|

当然我们都知道linux大部分的命令设置都是临时生效，而且ulimit命令只对当前终端生效

如果需要永久生效的话，我们有两种方法，

1. 一种是将命令写至profile和bashrc中，相当于在登陆时自动动态修改限制

2. 还有一种就是在/etc/security/limits.conf中添加记录（需重启生效，并且在/etc/pam.d/中的seesion有使用到limit模块）

limits.conf文件附录
在/etc/security/limits.conf修改限制的格式如下

`domino type item value`
|参数|描述|
|----|----|
|domino|是以符号@开头的用户名或组名，\*表示所有用户|
|type|设置为hard or soft|
|item|指定想限制的资源。如cpu,core nproc or maxlogins|
|value|是相应的|

查看限制

{% highlight bash %}

➜  ~ ulimit -a
-t: cpu time (seconds)              unlimited
-f: file size (blocks)              unlimited
-d: data seg size (kbytes)          unlimited
-s: stack size (kbytes)             8192
-c: core file size (blocks)         0
-m: resident set size (kbytes)      unlimited
-u: processes                       62773
-n: file descriptors                1024
-l: locked-in-memory size (kbytes)  16384
-v: address space (kbytes)          unlimited
-x: file locks                      unlimited
-i: pending signals                 62773
-q: bytes in POSIX msg queues       819200
-e: max nice                        0
-r: max rt priority                 0
-N 15:                              unlimited

{% endhighlight %}

# 2. 最大打开文件数

### file-max系统最大打开文件描述符数

------

/proc/sys/fs/file-max中指定了系统范围内所有进程可打开的文件句柄的数量限制(系统级别, kernel-level).

> The value in file-max denotes the maximum number of file handles that the Linux kernel will allocate）.

> 当收到”Too many open files in system”这样的错误消息时, 就应该曾加这个值了.

对于2.2的内核, 还需要考虑inode-max, 一般inode-max设置为file-max的4倍. 对于2.4及以后的内核, 没有inode-max这个文件了.

### 查看实际值

------

可以使用cat /proc/sys/fs/file-max来查看当前系统中单进程可打开的文件描述符数目 

{% highlight bash %}

➜  ~ cat /proc/sys/fs/file-max
1602602

{% endhighlight %}

### 设置

------

- 临时性

> echo 1000000 > /proc/sys/fs/file-max

- 永久性：在/etc/sysctl.conf中设置

> fs.file-max = 1000000

### nr_open是单个进程可分配的最大文件数

内核支持的最大file handle数量，即一个进程最多使用的file handle数

> the maximum number of files that can be opened by process。A process cannot use more than NR_OPEN file descriptors.

一个进程不能使用超过NR_OPEN文件描述符。

{% highlight bash %}

➜  ~ cat /proc/sys/fs/nr_open 
1048576

{% endhighlight %}



**注： 原文：https://blog.csdn.net/gatieme/article/details/51058797** 