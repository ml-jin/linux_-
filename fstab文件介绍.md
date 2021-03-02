# Linux自动挂载（配置/etc/fatab）详解

[TOC]

## 了解了 mount 命令之后，读者可能会问，系统如何在开机时自动挂载硬盘，它又是怎么知道哪些分区是需要挂载的呢？

很简单，Linux 通过 /etc/fstab 配置文件来确定这些信息，这个配置文件对所有用户可读，但只有 root 用户有权修改此文件。也就是说，如果我们想实现开机自动挂载某个硬件设备，只需要使用 root 身份在 /etc/fstab 文件中添加此设备即可。

[root@localhost ~]# vi /etc/fstab
UUID=c2ca6f57-b15c-43ea-bca0-f239083d8bd2 / ext4 defaults 1 1
UUID=0b23d315-33a7-48a4-bd37-9248e5c44345 /boot ext4 defaults 1 2
UUID=4021be19-2751-4dd2-98cc-383368c39edb swap swap defaults 0 0
\#只有这三个是真正的硬盘分区，下面的都是虚拟文件系统或交换分区
tmpfs /dev/shm tmpfs defaults 0 0
devpts /dev/pts devpts gid=5, mode=620 0 0
sysfs /sys sysfe defaults 0 0
proc /proc proc defaults 0 0

目前，大家可以忽略 tmpfs、devpts、sysfs 和 proc 这几行，它们分别是与共享内存、终端窗口、设备信息和内核参数相关联的特殊设备。
可以看到，在 fstab 文件中，每行数据都分为了 6 个字段，它们的含义分别是：下面，我们一一进行讲解。首先介绍第一个字段，什么是 UUID 呢？UUID 即通用唯一标识符，是一个 128 位比特的数字，可以理解为就是硬盘的 ID，UUID 由系统自动生成和管理。

那么，每个分区的 UUID 到底是什么呢？用 dumpe2fs 命令（后续会讲）就可以查看到，具体执行命令如下：另外，也可以通过查看每个硬盘UUID的链接文件名来确定UUID，命令如下：
[Linux挂载](http://c.biancheng.net/view/2859.html)[Linux mount命令](http://c.biancheng.net/view/885.html)
第三个字段为文件系统名称，CentOS 6.3 的默认文件系统应该是 ext4。

第五个字段表示“指定分区是否被 dump 备份”，0 代表不备份，1 代表备份，2 代表不定期备份。

## 配置 /etc/fatab 文件

能看懂这个文件了吧？我们把 /dev/sdb5 和 /dev/sdb6 两个分区加入 /etc/fstab 文件，执行命令如下：

[root@localhost ~]# vi /etc/fstab
UUID=c2ca6f57-b15c-43ea-bca0-t239083d8bd2 ext4 defaults 1 1
UUID=0b23d315-33a7-48a4-bd37-9248e5c44345 I boot ext4 defaults 1 2
UUID=4021be19-2751-4dd2-98cc-383368c39edb swap swap defaults 0 0
tmpfs /dev/shm tmpfs defaults 0 0
devpts /dev/pts devpts gid=5, mode=620 0 0
sysfs /sys sysfs defaults 0 0
proc /proc proc defaults 0 0
/dev/sdb5 /disk5 ext4 defaults 1 2
/dev/sdb6 /disk6 ext4 defaults 1 2

可以看到，这里并没有使用分区的 UUID，而是直接写入分区设备文件名，也是可以的。不过，如果不写 UUID，就要注意，在修改了磁盘顺序后，/etc/fstab 文件也要相应的改变。
至此，分区就建立完成了，接下来只要重新启动，测试一下系统是否可以正常启动就可以了。只要 /etc/fstab 文件修改正确，就不会出现任何问题。