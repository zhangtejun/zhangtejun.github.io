---
layout: post
title:  "linuxShell2"
date:   2017-07-08 10:36:00
author: zhangtejun
categories: zhangtejun
---
##### shell 分为CLI与GUI
壳层（Shell）在计算机科学中，是指“提供用户使用界面”的软件，通常指的是命令行界面的解析器。
一般来说，这个词是指操作系统中，提供访问内核所提供之服务的程序。

* 图形界面shell（Graphical User Interface shell 即 GUI shell）

* 命令行式shell（Command Line Interface shell ，即CLI shell）

用户不能直接操作Kemel，所以需要通过Shell来操作Kemel（内核）。
##### BASH
1. 命令一般由命令，选项，参数组成。

2. -a 与--all一样

3. history 查看所有历史记录命令。

4.  Ctrl+ r  在历史记录里搜索命令。包含任意字符。
    从新调用前一个命令中的参数。按下esc后按.键。
    
5. su -(全新的终端加-) 切换到root 用户。sudo 使用管理员身份运行命令。

6. 修改密码 passwd

7. jobs 显示后台作业（bg 编号 改变后台工作状态）（fg 编号 后台拉回前台）

8. Ctrl+C  终止命令。 命令+&  后台运行 如：firefox &

9. 命令行通配符：* 匹配0个或多个。
	？ 匹配任意一个。
	`[0-9] `匹配一个数字范围。
	`[abc]` 匹配列表里任意字符。
	`[^abc]` 匹配列表以外字符。
	
##### 文件系统结构

1. 当前工作目录（pwd查看目录）
	每个shell或进程都有一个当前目录
	
2. 通过touch命令可以创建一个空白文件或更新已有文件的时间。

3. 以“.”开头的文件为隐藏文件。

4. ls -a (显示所有文件，包括隐藏文件) -l(详细信息)
   ls -R(所有子目录) ls -ld(显示目录信息)
   
5. 绝对路径与相对路径
* 绝对路径：  以“/“开头，递归每级目录直到目标的路径。不受当前所在工作目录限制。
* 相对路径： 以当前目录为起点，到达目标所在的路径。收当前所在目录的限制。

6.cd命令切换目录

上一级目录".."  当前目录"." 用户家目录"~" 上一工作目录"-"

#####  Linux文件基本操作管理

1. 使用cp命令复制文件或目录
`格式   cp （-r）源文件（文件夹） 目标文件（文件夹参数-r(文件夹与里面文件) （-v显示详细信息） -r -v与-rv一样）`
2. 移动，重命名（在当前文件夹 移动）
`格式    mv 当前文件夹  目标文件夹或目标文件夹/别名（移动并重命名）`

3. 创建，删除（文件，文件夹）
     ` rm  touch`
    
    `rm 删除文件（不能删除文件夹）`
    
    `rm -r 不提示直接删除文件夹和文件`
    
    `rm -ri 逐个删除 并提示`
    
    `rm -rif 不会提示 强制删除`

4. 创建(mkdir)，删除目录(rmdir  不能删除非空文件夹)

    `rmdir -r 删除非空文件夹`
##### Linux系统常用命令
**up**
1. 日期时间
date 查看当前系统时间日期，格式化显示时间：+%Y--%m--%d
date -s "20150909" 修改日期    date -s "20：09：08" 修改日期
2. hwclock(clock) 显示硬件时钟时间
3. cal用以查看日历
4. 命令uptime 查看系统运行时间   man uptime查看系统负载

5. 输出查看
6. echo "输入的内容" 显示输入的命令
7. cat 文件名 显示文件内容  head 显示文件的头几行（默认10）-n指定显示行数
8. tail 文件名 显示文件的末几行（默认10）-n指定显行数  -f 追踪显示文件更新（一般用于查看日志，命令不会退出而是持续显示新加入的内容）
9. more 文件名  用于翻页显示文件内容（只能向下翻 空格键翻页）
10. less 文件名  用于翻页显示文件内容（带上下翻页 空格键翻页）

**查看硬件信息**
1. lspci 查看PCI设备   -v查看详细信息
2. lsusb 查看USB设备   -v查看详细信息
3. lsmod 查看加载的模块（驱动）

**关机，重启**
1. shutdown [关机，重启] 时间   ---用以关闭，重启计算机

	立即关机：shutdown -h now
	
	10分钟后关机：shutdown -h +10
	
	23：30 关机：shutdown -h 23：30
	
	立即重启：shutdown -r now
	
2. 命令poweroff立即关闭计算机
         reboot立即重启计算机

**归档，压缩**
    ```shell
    1. zip压缩文件
    
    zip linuxcast.zip(压缩后文件名)     myfile(压缩文件)
    
    2. unzip 解压zip文件
    
      upzip linuxcast.zip(压缩后文件名)  myfile(压缩文件)
    
    3. gzip压缩文件
    
      gzip linuxcast.zip(压缩后文件名)
      
    4. tar 用于归档文件（不压缩）
    
      tar -cvf linuxcast.tar(归档后文件名) myfile(想要归档文件/文件夹）
      
      tar -xvf linuxcast.tar(归档后文件名) 
      
      tar -cvzf linuxcast.tar.gz /etc(代表文件夹)  (归档后再压缩 实际数调用gzip)
    ```	

**查找**
1. locate 快速查找文件，文件夹

	locate keyword（关键字 缺点 数据库更新时间长）
    此命令需要预先建立数据库，数据库默认每天更新一次，可
    用updatedb手工创建，更新数据库。
    
2. find 用以高级查找文件，文件夹（缺点 查询时间长）
   ```shell
   find . -name *linuxcast*    (.代表当前目录，-name 以文件名查找 ×0个或以上)
   
   find / -perm 777  根分区下 所有权限为777的
   
   find / -type d     查找所有的目录。  -type l 所有的链接。
   
   find . -name "a*" -exec ls -l {} \;    解释：find . -name "a*" -exec  {} \；是固定格式。查找所有以a开头的文件，并显示其信息。
   ```
   
   
##### Vi 文本编译器

1. vim 
	一般通过vim+ 目标文件 的形式使用vim，如果目标文件存在则vim打开该文件，如果目标文件不存在，则vim新建并打开该文件。
2. vim的三种模式：
	* 命令模式（常规模式）
	    vim启动后，默认进入命令模式，任何模式都可以通过esc键回到命令模式（可以多按几次)。命令模式下 可以通过键入不同的命令完成选折，复制，粘贴，撤销等等操作。
	* 插入模式
	    在命令模式中按“i”键即可进入插入模式， 在插入模式下可以输入编辑文本内容，使用esc键可以回到命令模式。
	* ex模式
	    在命令模式按“：”键进入ex模式，光标会移动到底部，在这里可以保存修改或退出vim。

3. 命令模式下

    i   在光标前插入文本
     
     o   在当前行的下面插入新行
     
     dd  删除整行
     
     yy  将但前行的内容放入缓冲区（复制当行）
     
     n加yy 将n行的内容放入缓冲区（复制n行）
     
     p   将缓冲区的文本放入光标后（粘贴）
     
     u   撤销上一个操作。
     
     r   替换当前字符
     
     /   查找关键字  在切换出来厚n切换关键字。

4. EX模式
	命令模式下按“：”键可以进入ex模式 ，光标移动到底部，此时可以修改或保存或退出Vim
	
	:w     保存当前修改
	
	:q     退出
	
	:q!     强制退出，不保存修改。
	
	:x      保存并退出，相当于 :wq
	
	:wq     保存并退出，相当于 :x
	
	:set number 显示行号
	
	:!系统命令  执行一个系统命令并显示结果
	
	:sh        切换到命令行，使用ctrl+d返回vim
	

##### 磁盘基本概念
 1. 磁盘的基本概念
 
	cylinder（柱面）
	
	sector（扇区）
	
	head（磁头）512字节，
	
 2. Linux所以设备都被抽象为一个文件，保存在/dev目录下。
 
	`设备的名称一般为hd[a-z]或sd[a-z]([a-z]为分区号)，如：hda,hdb,sda,sdb`
	
	`IDE设备的名称为hd[a-z],SATA,SCSI,SAS,USB等设备为sd[a-z]`
 
 3. 分区
 
	将磁盘分几个区，以方便管理。
	
	不同分区：设备名称+分区号 方式表示，如sda1,sda2。
	
	主流的分区机制分为MBR和GPT两种。
	
	MBR是传统的分区机制，应用与绝大多数使用BIOS的PC设备。
	
	MBR支持32bit和64bit
		
	MBR支持的分区数量有限。
		
	MBR只支持不超过2T的硬盘，超过2T的硬盘将智能使用2T空间（有第三方决绝方法）
		
	MBR分区
	
	* 主分区
		
		  最多只能创建4个主分区。mbr结构只有4个条目
			
	* 扩展分区
		
		一个扩展分区会占用一个主分区的位置
			
	* 逻辑分区
		
		Linux最多支持63个IDE分区和15个SCSI分区。
	
	GPT分区
	
	决绝了MBR的很多缺点
		
	支持超过2T的分区
	
	向后兼容MBR
	
	必须在支持UEFI（Intel提出的）的硬件才能使用
	
	必须使用64bit系统
	
	Mac，Linux系统都支持GPT分区格式
	
	window7 64bit，windowServer2008 64bit支持GPT
	
##### 使用fdisk进行磁盘管理

1. fdisk是来自IBM的老牌分区工具，支持绝大多数操作系统，
	几乎所以的Linux的发行版本都装有fdisk，包括在Linux的rescue模式下的依然能够使用。
	fdisk是一个基于MBR的分区工具，所以如果需要使用GPT，则无法使用fdisk进行分区
	
2. FDISK（逻辑分区从5开始）
	
    fdisk命令只有具有超级用户权限才能够运行
    
    使用fdisk -l可以列出所有安装的磁盘及其分区信息
    
    使用fdisk/dev/sda可以对目标磁盘进行分区操作
    
    分区之后需要使用partprobe命令让内核更新分区信息、
    
    否则需要重启才能识别新的分区。
    
    /proc/partitions文件也可以用来查看分区信息。
		
##### Linux文件系统
1. 文件系统
  操作系统通过文件系统管理文件及数据，磁盘或分区需要创建文件系统
  之后才能够为操作系统使用，创建文件系统的过程又称之为格式化。
  没有文件系统的设备称之为裸设备
  常见的文件系统有fat32，NTFS,EXT2...4,xfs,HFS等。
  
    文件系统间的区别：日志，支持分区大小，支持单过文件大小，性能等
  
    window主流为NTFS
  
   Linux下主流文件系统：Ext3,Ext4
  
2. MKE2FS
    ```
      命令mke2fs用来创建文件系统
        mke2fs -t ext4 /dev/sda3
        常用参数：
        -b blocksize 指定文件系统块大小
        -c 			  建立文件系统时检查坏损块
        -L lable      指定卷标
        -j            建立文件系统日志
    ```
3. MKFS
  命令mkfs也可以创建文件系统，先对于mke2fs简单，但是支持的参数少
  
    mkfs.ext4 /dev/sda3
  
4. DUMPE2FS

    命令dumpe2fs可以用来查看分区的文件建系统信息
    
    dumpe2fs /dev/sda3
    
5. journal 日志
  带日志的文件系统（ext3,ext4) 拥有较强的稳定性，在出现错误时可以进行恢复。
  
    使用带日志的文件系统，文件系统会使用一个叫“两阶段提交”的方式进行磁盘操作
  ，当磁盘操作时，文件系统进行以下操作：
  
    1 文件系统将准备执行的事物的具体内容写入日志
    
    2 文件系统进行操作
    
    3 操作成功后，将事物的具体内容从日志中删除
    
   好处：当事务执行的时候如果出现了意外（断电，磁盘故障等），可以通过查询日至进行恢复操作
  
   缺点：丧失一定的性能（额外的日志读写操作）。
6. E2LAbLE（便签 方便管理）
    
    命令e2lable可以用来为文件系统添加标签
    
    e2lable /dev/sda2 显示sda2的系统标签
    
    e2lable /dev/sda2 LINUXCAST 将sda2的系统标签设置为LINUXCAST。

7. FSCK
    命令fack用来检查并修复损坏的文件系统
    
    fack /dev/sda2
    
    使用-y参数不提示而直接修复
    
    默认fack会自动判断文件系统类型，如果文件系统损坏较严重，请使用-t参数指定文件系统的类型。
    
    对于识别为文件的损坏数据（文件系统无记录），fack会将该文件放入lost+found目录
    
    系统启动时会对磁盘进行fack操作。

##### Linux文件系统挂载管理（修改文件系统一定要卸载磁盘）

1. 挂载操作
  
    磁盘或分区创建好文件系统后，需要挂载到一个目录才能够使用。
    windows或Mac系统会进行自动挂载，一旦创建号文件系统后会自动挂载到系统上，windows上称C盘等。
    Linux需要手工进行挂载操作或配置系统进行自动挂载。
    
    `/dev/sda3 ext4  --挂载-->/mnt`

2. MOUNT

    在linux中，我们通过mount命令将格式化好的磁盘或分区挂载到一个目录上。
    
    mount /dev/sda1(要挂载的分区) /mnt/(挂载点)
  
    常用参数：

    不带参数的mount命令会显示所有已挂载的文件系统
    
    -t 指定文件系统的类型
    
    -o 指定挂载的选项
    
      ro ,rw 以只读或读写的形式挂载，默认是rw
      
      sync 代表不使用缓存，而是对所有操作直接写入磁盘
      
      async 代表使用缓存，默认是async.
      
      noatime代表每次访问文件时不更新文件的访问时间
      
      atime代表每次访问文件时不更新文件的访问时间
      
      remount 重新挂载文件系统

  3. UMOUNT
  
    ```shell
    命令umount用来卸载已经挂载的文件系统，相当于windows中的弹出
    
    umount 文件系统/挂载点
    
    umount /dev/sda3     或者 umount /mnt
    
    如果出现device is busy报错，则表示该文件系统正在被使用，无法卸载可以使用以下命令查看使用文件系统的进程：
    
    fuser -m/mnt
    
    也可以使用命令lsof查看正在被使用的文件：
    
    lsof/mnt
    ```

4. 自动挂载
   配置文件/etc/fstab用来定义需要自动挂载的文件系统，fstab中每一行代表一个挂载配置，格式如下：
   
    `/dev/sda3       /mnt   ext4      defaults   0 0`
    
    需要挂载的设备   挂载点 文件系统   挂载选项   dump,fsck相关选项
    
    要挂载的设备也可以使用LABEL进行识别，使用LABEL=LINUCAST取代/dev/sda3
    
    mount -a 命令会挂载所有fatab中定义的自动挂载。

##### Linux下获取帮助
  
  1. 几乎所有的命令都可以使用-h或--help参数获取使用方法，参数信息等。
    
      --help列出的信息比较简单，此时用man查看更详细的参数。
      
       man(手册缩写）命令是Linux中最常用的帮助命令，将要获取帮助的命令作为参数运行man命令就可以获取相应的文档帮助。
      
      如：man ls
      
      man -k 关键字   可以用来查询包含该关键字的文档。如：man -k  pass  所以包含*pass*的都会被列出来。
      
      info与man类似，但是提供更详细的信息，以类似网页的形式显示
      
      man与info 都可以通过“ /+关键字 ”的方式进行搜索。
      
      很多程序，命令都带有详细的文档，以TXT,html,pdf等方式保存在/usr/share/doc目录中。是相应程序最为详尽的文档。
      
##### Linux用户基础
1. **用户，组**
		当我们使用Linux时，需要以一个用户的身份登入，一个进程也需要以一个用户的身份运行，用户限制使用者或进程可以使用，不可以使用哪些资源。
	组用来方便组织管理用户
		每一个用户拥有一个UserID，操作系统实际使用的是用户ID，而非用户名。
		每一个用户属于一个主组，属于一个或多个附属组。
		每一个组拥有一个GroupID.
		每个进程以一个用户身份运行，并受该用户可访问的资源限制。
		每个可登陆用户拥有一个指定的shell

	**用户**
		用户ID为32位，从0开始，但是为了和老式系统（16位）兼容，用户ID限制在60000以下。
		用户分为一下三类：
		Root用户  (ID为0的用户为root用户)
		系统用户  (1~499) 没有shell
		普通用户  (500以上)
		系统中的文件都有一个所属用户及所属组
		使用id命令可以像是当前用户的信息。
		使用password命令可以修改当前用户密码。
		
	**相关文件**	
	
      /etc/passwd  -保存用户信息
      
      /etc/shadow  -保存用户密码（加密的）
      
      /etc/group   -保存组信息
		
	**查看登录的用户**
	
      命令whoami显示当前用户
      
      命令who显示有哪些用户已经登陆系统
      
      命令w显示有哪些用户已经登录并且在干什么。
	
	**创建一个用户**
	
      命令useradd用以创建一个新用户
      
      useradd nash_su
      
      这个命令会执行以下操作：
      
      1 在/etc/passwd中添加用户信息
      
      2 如果用户使用passwd命令创建密码。则将密码保存在/etc/shadow中（passwd nash_su）
      
      3 为用户创建一个新的家目录/home/nash_su
      
      4 将.etc/skel中文件复制到家目录中    
      
      5 建立一个与用户名相同的组，新建用户默认属于个组
      
      命令useradd支持以下参数
     
      -d 指定家目录
      
      -s 登录shell
      
      -u userid
      
      -g 主组
      
      -G 附属组（最多31个，用“，”分割
      
      也可以通过直接修改/etc/passwd的方式实现，但是不建议
        
修改用户信息
```shell
命令usermod用来修改用户信息
usermod 参数 username
命令usermod支持以下参数
-l 新的用户名
-u 新的userid
-d 用户家目录位置
-g 用户所在主族
-G 用户所属附属组
-L 锁定用户使其不能登录路
-U 解除锁定
```
		
删除用户	
```shell
命令userdel用以删除指定的用户
userdel nash_su (不会删除用户家目录)
userdel -r nash_su (同时删除用户家目录)
```
组
	
几乎所有的操作系统都有组的概念。通过组，我们可以更加方便的归类，管理用户
一般来讲，我们使用部门，职能或地理区域的分类方式来创建使用组。
每个组都有一个组id
组信息保存在/etc/group中
每个用户拥有一个组，同时还可以拥有最多31个附属组
```shell		
创建，修改，删除组
命令groupadd用以创建组
命令groupmod -n/-g newname/newGid oldname/oldGid  修改组名/组id
命令groupdel 删除组
```
##### Linux权限机制
* 权限
    权限是操作系统用来限制对资源的访问的机制，权限一般分为读，写，执行。系统中每个文件都拥有特定的权限，所属组及
    所属用户，通过这样的机制来限制哪些用户，那些组可以对特定文件进行什么样的操作。
    
    每个进程都是一某个用户的身份运行，所以进程的权限与用户权限一样，用户的权限大，该进程拥有权限就大。
* 文件权限

  ```shell
    权限              对文件的影响         对目录的影响
    
    r(读取)           可读取文件内容        可列出目录内容
    
    w(写入)           可修改文件内容        可在目录中创建删除文件
    
    x(执行)           可以作为命令执行       可访问目录内容
  ```
   注意：目录必须拥有X权限，否则无法查看其内容。

    Linux权限基于UGO模型进行控制：
    ```shell
      U代表User，G代表Group，O代表Other
      每一个文件的权限基于UGO进行设置
      权限3个一组（rwx）,对应UGO分别设置
      每一个文件拥有一个所属用户和所属组，对应UG，不属于该文件所属用户或所属组的使用O权限
	  ```
	命令ls -l可以查看当前目录下文件的详细信息：
	
    ```javascript
    drwxr-xr-- 2 nash_su training 208 Oct 1 13:50 linuxcast.nets
    ```
	
   drwxr-xr--   d代表文件类型是一个目录，-是普通文件，l是链接等
	
* 修改文件所属用户，组
	命令chown用以改变文件的所属用户：
	
	chown nash_su linuxcast.net
		
    -R参数递归的修改目录下的所有文件的所属用户
  
    命令chgrp用以改变文件的所属组
    
    chgrp nash_su linuxcast.net
    
    -R参数递归的修改目录下的所有文件的所属组
		
* 修改权限
  ```shell
  命令chmod用以修改文件的权限
      
     chmod 模式 文件
        
     模式为如下格式：
      
     u,g,o分别代表用户，组和其他
     a可以代表ugo
     +,-代表加入或删除对应权限
     r,w,x代表三种权限
    
     模式示例：
    
     chmod u+rw linuxcast.net
     chmod g-x linuxcast.net
     chmod o+r linuxcast.net
     chmod a-x linuxcast.net
      
    递归修改文件夹内的权限
    
    chmod -R a-x linuxcast.net
    
    命令chmod也支持数字方式修改权限	，三个权限分别由三个数字表示：
    
    -r =4 (2^2)
    
    -w =2 (2^1)
    
    -r =1 (2^0)
    
    没有权限就为0
    
    使用数字表示权限时，每组分别对应数字之和：
    
    rw =4+2 =6
    
    rwx=4+2+1 =7
    
    rw =4+1 =5
    
    所以，使用数字表示ugo权限：ugo必须全部修改
    
    chmod 660 linuxcast.net   ==   rw-rw----
  ```    

      
##### Linux扩展权限
每个终端都拥有一个umask属性，来确定新建文件，文件夹的默认权限

umask使用数字权限方式表示：022
目录的默认权限是：777-umask 

文件的默认权限是：666-umask

一般，普通用户的默认umask是002，root用户的默认umask是022

也就是说，对于普通用户来讲：

新建文件的权限是：666-002 = 664

新建目录的权限是：777-002 = 775

(新建文件 touch 文件名)

命令umask用以查看设置（修改）umask值

umask 022

  root不受权限控制
  
特殊权限

  `suid`  `sgid`  `sticky`
  
三种特殊权限简介
* SUID

  当一个设置了SUID 位的可执行文件被执行时，该文件将以所有者的身份运行，也就是说无论谁来执
  行这个文件，他都有文件所有者的特权。
  如果所有者是 root 的话，那么执行人就有超级用户的特权了。

* SGID

  当一个设置了SGID 位的可执行文件运行时，该文件将具有所属组的特权， 任意存取整个组所能使用的系统资源。
  若一个目录设置了SGID，则所有被复制到这个目录下的文件， 其所属的组都会被重设为和这个目录一样，
  除非在复制文件时加上-p （preserve，保留文件属性）的参数，才能保留原来所属的群组设置。

* sticky-bit

  对一个文件设置了sticky-bit之后，尽管其他用户有写权限， 也必须由属主执行删除、移动等操作。
  对一个目录设置了sticky-bit之后，存放在该目录的文件仅准许其属主执行删除、 移动等操作。

设置特殊权限

设置suid:

```shell
chmod u+s linuxcast.net
```

设置sgid:
```shell
chmod g+s linuxcast.net
```
设置ticky:
```shell
chmod o+s linuxcast.net
```
与普通权限一样，特殊权限也可以使用数字方式表示

- SUID =4
- SGID =2
- STICKY =1

所以，我们可以通过以下命令设置：

```shell
chmod 4755 linuxcast.net
```

##### 网络基础
Ip编址
ip编址是一个双层编址方案，一个ip地址标识一个主机（或一个网卡接口）
现在最广泛的是ipv4，逐渐向ipv6切换
ipv4地址为32位，ipv6地址为128位长.
	
一个ip地址分为网络部分和主机部分
网络部分用来标志所属区域，主机部分用来标识该区域的哪个主机。
ip地址
  ip地址共32位，通常用点分十进制表示。
  
  A类IP地址000
  
  B类IP地址100
  
  ......
  
  
子网掩码	
  子网掩码只有一个作用，就是将某个IP地址划分成网络地址和主机地址两部分。
  子网掩码的长度也是32位，左边是网络位，用二进制数字“1”表示，1的数目等于
  网络位的长度；右边是主机位，用二进制数字“0”表示。
  
**同一个网络主机间的通信（mac地址）**
  1 ARP(地址解析协议)消息，所以设备都收到，目标主机才会返回自己的mac地址。

不同网络间的通信（路由器/网关/具有路由功能的主机）
必须通过路由器转发。
  
**路由**

  在不同网络间传输数据的功能叫做路由功能，一般有多个接口，连接到不同的网络中
  并且通过路由表进行数据转发。

**域名**

  ip地址难记，所以用域名进行管理

  域名分为三个部分，用.分割。类型 域名 主机名  

  www(主机名).baidu（域名）.com（类型）

**DNS**

每个域名代表一个ip，而DNS服务就是将 ip与域名之间进行转换。

##### Linux网络基础配置
以太网连接

在Linux中，以太网接口被命令为：eth0,eth1等0，1代表网卡编号

通过lspci命令查看网卡硬件信息（如果是usb网卡，可能需要使用lsusb)

命令ifconfig命名用来查看接口信息

ifcongfig -a 查看所有接口

ifcongfig eth0 查看特定接口
		
命令ifup,ifdown用来启用，禁用一个接口

ifup eth0

ifdown eth0

配置网络信息

使用setup命令可以配置网络信息
		1 network。。。 2 默认第一个选项 3 选择网卡
		配置完成后需要使用ifup启用网卡，并且使用ifconfig命令查看信息
	网络相关配置文件
	
**网卡配置文件**
	
/etc/sysconfig/network-scripts/ifcfg-eth0

**DNS配置文件**

/etc/resolv.conf

**主机名配置文件**

/etc/sysconfig/network

**静态主机配置文件**

/etc/hosts

网络测试命令

网络连通 ping 192.168.3.3/www.gjfj.com

测试dns解析 host/dig www.gjfj.com

显示路由表 ip route

追踪到目标地址的网络路径 tracroute www.gjfj.com

使用mtr进行网络质量测试（结合ping,tracroute）

mtr www.gjfj.com

修改主机名

hostname

故障排查

从底层到高层，从自身到外部

先查看网络配置信息

查看网关是否联通 ping 网关

查看DNS解析是否正常
	
##### Linux多命令协作：管道及重定向
**管道和重定向**
在Linux系统中，大多数命令都很简单，很少出现复杂功能的命令，每个命令往往只
实现一个或几个简单的功能，我们可以通过将不同功能的命令组合在一起使用，已达到完成某个复杂功能目的
linux中，几乎所以的命令的返回数据都是纯文本的（因为命令都是运行在CLI下
），而纯文本形式的数据又是绝大多数命令的输入格式，这就让多命令协作成为可能。
Linux的命令行为为我们提供了管道和重定向机制，多命令协作就是通过管道和重定向完成的
	
命令行shell的数据流有以下定义：

  STDIN   标准输入   0    键盘
  
  STDOUT   标准输出   1   终端
  
  STDERR   标准错误   2    终端
  
	
命令通过STDIN接收参数或数据。通过STDOUT输出结果或STDERR输出错误。

管道和重定向我们可以控制CLI的数据流
```shell

分类	    关键字          定义                     例子

            >    将STDOUT重定向到文件（覆盖）  echo“linuxcast.net”>outfile 结果是linuxcast.net保存在文件outfile中

重定向	    >>   将STDOUT重定向到文件（追加）
 
            2>   将STDERR重定向到文件（覆盖）  比如报错信息
 
            2>&1 将STDOUT与STDERR结合              
 
            <    重定向到STDiN     grep linuxcast < /etc/passwd   把 /etc/passwd作为命令标准输入  输入给grep linuxcast
     
管道		    |	  将一个命令的STDOUT（标准输出）作为另一个命名的标准输入  find / -user linuxcast | grep Video
                查找根分区下所有属于linuxcast用户 的文件。文件名包含Video关键字。

```
##### Linux命令行文本处理工具
**文件浏览**

cat 查看文件内容

more 以翻页形式查看文件内容（只能向下翻页）

less 以翻页形式查看文件内容（可向上向下翻页）

head  查看文件开始10行（或指定行数）

tail  查看文件结束10行（或指定行数）

**基于关键字搜索**

命令grep用以基于关键字搜索文本（正则表达式命令）

`grep linuxcast /etc/passwd -----> 搜索/etc/passwd下的包含linuxcast的文件`

`find / -user linuxcast | grep Video  ----->所以属于user用户的文件 包含Video关键字`

`find / -user linuxcast 2> /dev/null | grep Video  ----->所以属于user用户的文件 包含Video关键字 将输出错误信息不显示出来。`-i 搜索时不区分大小写

-n 显示结果所在的行数

-v 输出不带关键字的行

-Ax 在输出的时候包含结果所在行之后的指定行数 x代表数字

-Bx 在输出的时候包含结果所在行之前的指定行数

**基于列处理文件（默认7个部分以 ：分割）**

命令cut 用以基于列处理文件内容
```shell
  cut -d: -f1 /etc/passwd 以:为分割符 将文件分为一列一列的,之后显示其中的第一列。
  grep linuxcast /etc/passwd | cut -d: -f3
```
-d指定分隔符（默认TAB）

-f指定输出列号

-c基于字符进行切割
```shell
cut -c2-6 /etc/passwd  第2到第6个字符
```
**文本统计**

 命令wc用以统计文本信息
 
  wc linuxcast
  
  -l 只统计行数
 
 -w 只统计单词
 
 -c 只统计字节数
 
 -m 只统计字符数
   
**文本排序**

命令sort对文本内容进行排序

```shell
sort linuxcast
```

  -r 倒序排序

  -n 基数排序

  -f 忽略大小写

  -u 删除重复行

  -tc 使用c作为分隔符为列进行排序

  -kx 当基于指定字符分割排序为列时，指定哪列进行排序

**删除重复行**
```shell
  sort -u
```
uniq只能删除相邻的重复行

**文本比较**

diff 比较2个文件的区别

-i 忽略大小写

-b 忽略空格数量的改变

-u  统一显示比较信息

**拼写检查**

处理文本内容

  命令tr
 ```shell
 删除关键字 tr -d'TMD'<linuxcast
 转换大小写 tr 'a-z' 'A-Z'<Linuxcast
 ```
**搜索替换**

命令sed搜索替换
```shell
sed 's/linux/unix/g' linuxcast  查找Linux关键字替换为Unix  g代表替换多个（有几个替换几个）。
sed '1，50s/linux/unix/g' linuxcast  指定1到50行
sed -e 's/linux/unix/g' linuxcast -e  's/nash_su/unix/g' linuxcast 指定多个
sed -f sededit linuxcast  将sed '1，50s/linux/unix/g' linuxcast保存在文件sededit中 下次直接调用
```

##### Linux系统启动详解
**BIOS**(basic input output system)称为基本输入输出系统。一般保存在主板上的BIOS芯片中。
 
    计算机启动的时候第一个运行的就是BIOS，BIOS负责检查硬件并且查找可启动设备
    可启动设备在BIOS设置中进行定义，如ＵＳＢ，ＨＤ，ＣＤＲＯＭ

**ＭＢＲ**

    ＢＩＯＳ找到可启动设备后执行其引导代码
    引导代码为ＭＢＲ（５１２字节）的前４４６字节。

**ＧＲＵＢ**
  
    ＧＲＵＢ是现在Linux使用的主流引导程序
    可以用来引导几乎所有的操作系统
    Ｇｒｕｂ配置文件保存在／ｂｏｏｔ／ｇｒｕｂ目录中。
    ｇｒｕｐ的配置文件／ｂｏｏｔ／ｇｒｕｂ／ｇｒｕｂ．ｃｏｎｆ

**ｋｅｒｅｎｌ**

    ＭＢＲ的引导代码将负责找到并加载Linux内核
    Linux内核保存在／ｂｏｏｔ／ｖｍｌｉｎｕｚ－２．６．３２－２７９．．．．ｉｍｇ

**INIT**

    init是Linux系统中运行的第一个进程
    调用/etc/rc.d/rc.sysinit负责对系统进行初始化，挂在文件系统，并根据运行级别启动相应的服务。
    Linux运行级别：3 5 正常用的界面
    -0 挂机
    -1 单用户
    - 不带网络的多用户
    - 多用户
    - 未使用
    - XII图形化多用户模式
    -6 重新启动
    可以通过/etc/inittab来配置默认的运行级别。更多的配置在/etc/init里
    runlevel显示上一个，当前运行级别。上一个没有默认为n.
    命令init可以用以改变当前运行级别。如init 3
    多用户修改ROOT密码
    为内核传递参数“1”或”single“可进入单用户模式.在最后加1或single
    单用户模式下不启动任何服务
    单用户模式直接以root用户登录，并且不需要密码
    可以用password修改root密码  exit 退出后进行重新重启
    防止密码被修改 使用GRUB加密
    通过在grub.conf中启动加入以下参数对其进行加密：
    password --md5 $1fdgdgjdfkgkdgjriogteriotikfkdf
    加密后的密码可以通过grub-md5-crypt生成。
  
  
##### RPM软件包管理
源代码形式

		绝大多数开源软件都是直接以源代码形式发布
		源代码一般都会被打包成tar.gz的归档压缩文件
		源代码需要编译成二进制形式后才能够运行使用
源代码基本编译流程

		1 ./configure  检查编译环境，相关库文件以及配置参数并生成makefile
		2 make  对源代码进行编译，生成可执行文件
		3 make install 将生成的可执行文件安装到当前计算机中
		源代码形式的软件使用起来教麻烦，但是兼容性，可控性好。
		开源软件一般会大量使用其他软件的功能，所以开源软件会有大量的依赖关系。
		
RPM

		源代码缺点：操作复杂，编译时间长
		优点：可定制，适合所以系统
		为方便使用，Erik Troan Marc Ewing开发了RPM(redhet Package Mannage)
		PRM通过将源代码基于特定的系统编译为可执行文件，并保存依赖关系。来简化开源软件的安装管理、
		RPM设计目标：
		使用简单
		使用单一软件文件发布 .rpm
		可升级
		追踪软件依赖关系
		基本信息查询
		软件验证功能
		支持多平台

rpm软件命令规范：

	    Linuxcast-1.2.0-30.el6.i686.rpm   el6代表系统，i686代表32位 X86-64

rpm基础命令：

		安装软件：rpm -i software.rpm   i代表install  通常用-ivh
		卸载软件：rpm -e software
		升级形式安装：rpm -U software-new.rpm
		rpm支持通过http,ftp协议安装软件：
		rpm -ivh http://www.linuxcast.net/software.rpm
	可以加入一下参数：
		-v显示详细信息）
		-h显示进度条

RPM查询

		rpm -qa                  列出所有安装的rpm软件
		rpm -qf 软件名            查询目标文件属于哪个包
		rpm -qi 包名              查询指定已安装rpm软件的信息
        rpm -ql 包名             查询指定已安装rpm软件的信息
		rpm -qip software.rpm    查询未安装rpm文件的信息
		rpm -qlp software.rpm    查询未安装rpm文件包含的文件		

rpm验证

		导入密钥：rpm --import RPM-GPG-KEY-CentOS-6
		验证rpm文件：
			rpm -K software.rpm
		验证已安装的软件：
			rpm -V software
			
			
##### YUM软件管理

rpm软件包形式管理虽然方便，但是需要手动解决软件包的依赖关系，使用YUM来解决
	
		自动解决依赖关系
		可以对RPM进行分组，并基于组进行安装操作
		引入仓库（repo）的概念，支持多仓库
		配置简单
		使用yum安装一个rpm软件时，如果有依赖关系，会自动在仓库中查找安装依赖软件。
		仓库可以是本地（是一个文件夹），也可以通过http,ftp或nfs形式使用集中的，统一的网络仓库。

yum仓库

		仓库的配置文件保存在/etc/yum.repos.d/目录下，格式如下：
		[LinuxCast]
		name=This is LinuxCast.net rpm soft repo
		baseurl=http://www.linuxcast.net/yun/centos/6/i386/rpms
		enabled=1
		gpgcheck=1
		仓库可以使用file,http,ftp,nfs方式
		yum配置文件必须已.repo结尾
		一个配置文件可以保存多个仓库信息
		/etc/yum.repos.d/目录下可以保存多个配置信息。

YUM的基本操作

		yum install software-name   software-name代表软件名     安装指定软件
		yum remove software-name   卸载指定软件
		yum updata software-name	升级指定软件

YUM查询操作

		yum search keyword   基于关键字的搜索
		yum list(all|install|recent|updata ) 列出全部的。安装的，最近的，软件更新
		yum info packagename  显示制定软件信息  相当于rpm -qi 软件名
		yum whatprovides filename 查讯哪个rpm软件包含目标文件

创建YUM仓库

		1.将所有的rpm文件拷贝到一个文件夹中 linuxcast-yum
		2.通过rpm命令手动安装createpo软件 rpm -ivh createpo........rpm
		3.通命令createpo -v /rpm-directory  索引目录，完成后会在当前目录下创建一个repodata文件夹来保存索引  
		  -v 显示详细信息  /rpm-directory代表包含rpm文件的文件夹目录中，如果在当前目录，则只需要createpo .  .代表当前目录。  
		3.2 如果有分组信息 则运行命令时使用-g参数指定文件
			createpo -g /tmp/*comps.xml /rpm-directory
				CentOS/RHEL的分组信息保存在光盘的repodata/目录下，文件名以comps.xml结尾的xml文件
		4.在/etc/yum.repos.d/目录下创建配置文件，以.repo文件
			[LinuxCast]
			name=LinuxCast.net yum repo
			baseurl=file:///linuxcast-yum/   /linuxcast-yum/代表决定路径。
			enabled=1  默认开启
			gpgcheck=1
		5.yum clean all 清楚缓存信息  原因：每次对yum仓库进行安装或查询类命令会重建yum缓存
		yum list进行查看信息。
		
		
		
		
