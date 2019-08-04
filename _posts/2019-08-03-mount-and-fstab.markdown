---
layout: post
title:  "mount命令及/etc/fstab文件详解"
date:   2019-08-03 12:00:00
categories: linux
tags: mount fstab umount
image: /images/pic03.jpg
---

一、手动挂载设备

mount 挂载命令
{% highlight bash %}
格式：

mount [options] [-t fstype] [-o option] 设备 挂载点
常用选项 [ options ]：

常用选项：
     -t fstype(ext2、ext3、ext4、xfs、iso9660、smb等)
     -r: 只读挂载
     -w: 读写
     -L lable: 以卷标指定， LABLE=“label”
     -U UUID：以UUID指定挂载设备，UUID=“UUID”
     -a: 自动挂载所有（/etc/fstab文件中）支持自动挂载的设备
     --bind Dir1 Dir2 己经挂载了的文件，可以再次绑定其它目录上使用
     -n: 不更新/etc/mtab文件
     
-o options
        async: 异步I/O
        sync: 同步I/O
        noatime/atime: 建议noatime
        auto/noauto: 是否能够被mount -a选项自动挂载；
        diratime/nodiratime: 是否更新目录的访问时间戳；
        exec/noexec：是否允许执行其中的二进制程序；
        _netdev: 网络设备
        remount: 重新挂载
        acl: 启用facl

            # tune2fs -o mount-option 设备
            # tune2fs -o ^mount-option 取消
{% endhighlight %}

示例：

（1）挂载,默认不指定选项会自动附加如下属性

defaults(use default options: rw, suid, dev, exec, auto, nouser, and async)
eg.
root@T430:~#  mount /dev/sda5 /mnt/sda5

（2）查看挂载情况 
查看当前已经挂载的情况有多种方式， "mount -s","cat /etc/mtab"等
{% highlight bash %}
➜  ~ mount -s
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=8035072k,nr_inodes=2008768,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,noexec,relatime,size=1611540k,mode=755)
/dev/sdb3 on / type ext4 (rw,relatime,errors=remount-ro)
{% endhighlight %}



二、查看占用挂载设备的进程
     fuser -v  挂载点
     fuser -km 挂载点
当卸载一个文件系统时，如果出现如下信息，表示该设备有人正在使用。
{% highlight bash %}
root@T430:~#  umount /dev/sda5
umount: /mnt/sda5: device is busy.
        (In some cases useful info about processes that use
         the device is found by lsof(8) or fuser(1))
先查看

root@T430:~#  fuser -c /mnt/sda5
/mnt/sda5:           13796c
root@T430:~#  fuser -c /mnt/sda5/find/
/mnt/sda5/find/:     13796c
杀死正在使用该文件系统的用户，然后才能进行卸载

root@T430:~#  fuser -km /mnt/sda5
/mnt/sda5:           13796c
{% endhighlight %}

四、挂载配置文件/etc/fstab
{% highlight bash %}
root@T430:~#  cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Fri Jul 31 23:50:21 2015
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/vg_system-root /                       ext4    defaults        1 1
UUID=3a9c20f4-0cc2-4563-9e2c-d4833c1463c2 /boot                   ext4    defaults        1 2
/dev/mapper/vg_system-var /var                    ext4    defaults        1 2
/dev/mapper/vg_system-swap swap                    swap    defaults        0 0
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0

该文件分为六段各段的意义如下：
 设备     挂载点   文件系统类型  挂载选项   转储频度   自检次序
（1）要挂载的设备：
        设备文件、LABEL=, UUID=
（2）挂载点：
        swap没有挂载点，挂载点为swap
（3）文件系统类型
        ext2、ext3、ext4、xfs、nfs、smb、iso9660等
（4）挂载选项：多个选项间使用逗号分隔;
        async、sync、_netdev
        defaults（ rw,  suid, dev, exec, auto, nouser, async, and relatime.）
（5）转储频率：
         0：从不备份
         1：每日备份
         2：每隔一天备份
（6）自检次序：
         0: 不自检
         1：首先自检，通常只能被/使用；
         2：等数字为1的自检完成后，再进行自检
{% endhighlight %}

注意：配置完该文件不会立即生效，可以重启操作系统或使用mount -a来使该文件立即生效。