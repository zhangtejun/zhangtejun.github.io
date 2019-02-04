---
layout: post
title:  "Logical Volume（逻辑卷）"
date:   2017-10-04 20:44:21
author: zhangtejun
categories: linux
---
##### LVM
LVM是 Logical Volume Manager(逻辑卷管理)的简写，它由Heinz Mauelshagen在Linux 2.4内核上实现。
LVM将一个或多个硬盘的分区在逻辑上集合，相当于一个大硬盘来使用，当硬盘的空间不够使用的时候，
可以继续将其它的硬盘的分区加入其中，这样可以实现磁盘空间的动态管理，相对于普通的磁盘分区有很大的灵活性。

LVM它在硬盘的硬盘分区之上，又创建一个逻辑层，以方便系统管理硬盘分区系统。
LVM 可以将分区和磁盘聚合成一个虚拟磁盘（virtual disk），从而用小的存储空间组成一个统一的大空间。这个虚拟磁盘在 LVM 术语中称为卷组（volume group）。

* 物理存储介质（The physical media）
  这里指系统的存储设备：硬盘，是存储系统最低层的存储单元。
* PV，物理卷（Physical Volume），指硬盘分区或从逻辑上与磁盘分区具有同样功能的设备（如RAID），是LVM的基本存储逻辑块，
  但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数。
* VG，卷组（Volume Group），即 LVM 卷组，它可由一个或数个 PV 组成，相当于 LVM 的存储池。
* PE，物理扩展单元（Physical Extends），每个 PV 都会以 PE 为基本单元划分。
* LE，逻辑扩展单元（Logical Extends），组成 LV 的基本单元，一个 LE 对应一个 PE。
* LV，逻辑卷（Logical Volume），它建立在 VG 之上，文件系统之下，由若干个 LE 组成。

![Linux Logical Volume Manager (LVM) v1]({{ site.p9 | prepend: site.baseurl }})

可以看到，①物理卷（PV）被由大小等同的基本单元PE组成。②一个卷组由一个或多个物理卷组成。
③PE和LE有着一一对应的关系。逻辑卷建立在卷组上。逻辑卷就相当于非LVM系统的磁盘分区，可以在其上创建文件系统。

The physical volumes are combined into logical volumes, with the exception of the /boot/ partition. The /boot/ partition cannot be on a logical volume group because the boot loader cannot read it. If the root (/) partition is on a logical volume, create a separate /boot/ partition which is not a part of a volume group.


##### 物理到逻辑卷的映射
![物理到逻辑卷的映射]({{ site.p10 | prepend: site.baseurl }})
物理磁盘 0 上的所有四个分区（/dev/hda[1-4]）以及完整的物理磁盘 1（/dev/hdb）和物理磁盘 2（/dev/hdd）作为物理卷添加到卷组 VG0 中。
卷组是实现 n-to-m 映射的关键（也就是，将 n 个 PV 看作 m 个 LV）。在将 PV 分配给卷组之后， 就可以创建任意大小的逻辑卷（只要不超过 VG 的大小）。
在图的示例中，创建了一个称为 LV0 的卷组，并给其他 LV 留下了一些空间（这些空间也可以用来应付 LV0 以后的增长）。

LVM 中的逻辑卷就相当于物理磁盘分区；在实际使用中，它们就是 物理磁盘分区。
在创建 LV 之后，可以使用任何文件系统对它进行格式化并将它挂载在某个挂载点上，然后就可以开始使用它了。图 3 显示一个经过格式化的逻辑卷 LV0 被挂载在 /var。



![图 3. 物理卷到文件系统的映射]({{ site.p11 | prepend: site.baseurl }})
##### 区段
为了实现 n-to-m 物理到逻辑卷映射，PV 和 VG 的基本块必须具有相同的大小；这些基本块称为物理区段（PE）和逻辑区段（LE）。尽管 n 个物理卷映射到 m 个逻辑卷，但是 PE 和 LE 总是一对一映射的。

在使用 LVM2 时，对于每个 PV/LV 的最大区段数量并没有限制。默认的区段大小是 4MB，对于大多数配置不需要修改这个设置，因为区段的大小并不影响 I/O 性能。但是，区段数量太多会降低 LVM 工具的效率，所以可以使用比较大的区段，从而降低区段数量。但是注意，在一个 VG 中不能混用不同的区段大小，而且用 LVM 修改区段大小是一种不安全的操作，会破坏数据。所以建议在初始设置时选择一个区段大小，以后不再修改。

不同的区段大小意味着不同的 VG 粒度。例如，如果选择的区段大小是 4GB，那么只能以 4GB 的整数倍缩小或扩展 LV。

图 4 用 PE 和 LE 显示与前一个示例相同的布局（VG0 中的空闲空间也由空闲 LE 组成，尽管图中没有显示它们）。

![图 4. 物理到逻辑区段的映射]({{ site.p12 | prepend: site.baseurl }})
另外，请注意图 4 中的区段分配策略。LVM2 并非总是连续分配 PE；细节参见关于 lvm 的 Linux 手册页（见 参考资料 中的链接）。系统管理员可以设置不同的分配策略，但是一般不需要这么做，因为默认策略（名为一般分配策略（normal allocation policy））使用符合常规的规则，比如不把并行的条带放在同一物理卷上。

如果决定创建第二个 LV（LV1），那么最终的 PE 布局可能像图 5 这样。

LVM version 2, or LVM2, is the default for Red Hat Enterprise Linux 5, which uses the device mapper driver contained in the 2.6 kernel. LVM2 can be upgraded from versions of Red Hat Enterprise Linux running the 2.4 kernel.

![图 5. 物理到逻辑区段的映射]({{ site.p13 | prepend: site.baseurl }})

[逻辑卷管理](https://www.ibm.com/developerworks/cn/linux/l-lvm2/#resources)




分区数据盘

通过步骤三介绍的方法登录 Linux 云服务器。

注意：
仅支持对数据盘进行分区，不支持对系统盘进行分区。若您强行对系统盘分区可能导致系统崩溃等严重问题，针对此种情况腾讯云不承担赔偿责任。
输入命令fdisk -l查看您的数据盘信息。
本示例中，有一个 54 GB 的数据盘(/vdb)需要挂载。

注意：
fdisk -l与df -h 都为拆看数据盘信息命令，但在没有分区和格式化数据盘之前，使用df -h 命令无法看到数据盘。


对数据盘进行分区。按照界面的提示，依次操作：

输入fdisk /dev/vdb(对数据盘进行分区)，回车；
输入n(新建分区)，回车；
输入p(新建扩展分区)，回车；
输入1(使用第 1 个主分区)，回车；
输入回车(使用默认配置)；
再次输入回车(使用默认配置)；
输入wq(保存分区表)，回车开始分区。
这里以创建 1 个分区为例，开发者也可以根据自己的需求创建多个分区。


使用fdisk -l命令，即可查看到，新的分区 vdb1 已经创建完成。


格式化数据盘

新分区格式化
分区后需要对分好的区进行格式化，您可自行决定文件系统的格式，如 ext2、ext3 等。本例以 ext3 为例。
使用下面的命令对新分区进行格式化：

 mkfs.ext3 /dev/vdb1


挂载分区
使用以下命令创建 mydata 目录并将分区挂载在该目录下：

 mkdir /mydata
mount /dev/vdb1 /mydata
使用命令查看挂载：

 df -h
出现如图框选的 vdb1 信息则说明挂载成功，即可以查看到数据盘了。


设置启动自动挂载
如果希望云服务器在重启或开机时能自动挂载数据盘，必须将分区信息添加到 /etc/fstab中。
使用以下命令添加分区信息：

 echo '/dev/vdb1 /mydata ext3 defaults 0 0' >> /etc/fstab
使用以下命令查看：

 cat /etc/fstab
出现如图最下方框选的 vdb1 信息则说明添加分区信息成功。
