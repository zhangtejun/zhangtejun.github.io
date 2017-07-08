---
layout: post
title:  "linuxShell"
date:   2017-07-08 10:32:00
author: zhangtejun
categories: zhangtejun
---
##### 创建符号链接(软链接)和硬链接命令 ln
windows下有快捷方式，linux下对应的是符号链接(symbolic links),建立符号链接格式为：
`ln -s 目标文件(源文件) 链接文件` -s或\-\-symbolic是必选的。
例：建立一个名为sym_a.txt,目标文件为a.txt的符号链接。
```shell
root@zhtjun:~/weblogic# ln -s a.txt sym_a.txt
root@zhtjun:~/weblogic# ls -l *.txt
-rw-r--r-- 1 root root 7 Jul  8 10:38 a.txt
lrwxrwxrwx 1 root root 5 Jul  8 10:40 sym_a.txt -> a.txt
```
说明:sym_a.txt所在的第一列的第一个字符为1，1表示文件类型是符号链接。5表示字符串（目标文件）a.txt的长度。
**断链**：源文件不存在的情况下（命令可以执行成功）叫断链。

**删除符号链接文件不影响目标文件。**

也可以对`目录`建立符号链接：
```shell
root@zhtjun:~/weblogic# ln -s /root/weblogic/aa/ test
root@zhtjun:~/weblogic# ls -lh
total 4.0K
drwxr-xr-x 2 root root 4.0K Jul  8 10:47 aa
lrwxrwxrwx 1 root root   18 Jul  8 10:48 test -> /root/weblogic/aa/
```

##### 硬链接（源文件必须存在）
建立硬链接格式：
`ln 目标文件(已存在的源文件) 链接文件`

```shell
#创建文件a.txt >>追加方式       >替换方式
root@zhtjun:~/weblogic# echo 'test' >> a.txt
#复制文件a.txt
root@zhtjun:~/weblogic# cp a.txt a_copy.txt
#创建硬链接
root@zhtjun:~/weblogic# ln a.txt a_link.txt
root@zhtjun:~/weblogic# ls -l *
-rw-r--r-- 1 root root 5 Jul  8 10:58 a_copy.txt
-rw-r--r-- 2 root root 5 Jul  8 10:58 a_link.txt
-rw-r--r-- 2 root root 5 Jul  8 10:58 a.txt
#查看inode
root@zhtjun:~/weblogic# ls -li a*
426020 -rw-r--r-- 1 root root 5 Jul  8 10:58 a_copy.txt
426019 -rw-r--r-- 2 root root 5 Jul  8 10:58 a_link.txt
426019 -rw-r--r-- 2 root root 5 Jul  8 10:58 a.txt
```
可以看见a.txt和a_link.txt的硬链接数为2，再创建一个将为3。而复制的文件为1。

`ls -i或者\-\-inode`查看文件的inode。每一个文件或者目录对应一个inode。
a_link.txt或者(a.txt)的内容改变将互相影响。可以简单理解为它们互为硬链接。

```shell
root@zhtjun:~/weblogic# ls -li a*
426020 -rw-r--r-- 1 root root 5 Jul  8 10:58 a_copy.txt
426019 -rw-r--r-- 2 root root 4 Jul  8 11:06 a_link.txt
426019 -rw-r--r-- 2 root root 4 Jul  8 11:06 a.txt
root@zhtjun:~/weblogic# rm -rf a.txt 
#源文件a.txt 删除后，链接文件正常，并且链接文件数减1
root@zhtjun:~/weblogic# ls -li a*
426020 -rw-r--r-- 1 root root 5 Jul  8 10:58 a_copy.txt
426019 -rw-r--r-- 1 root root 4 Jul  8 11:06 a_link.txt
root@zhtjun:~/weblogic# cat a_link.txt 
```
* 硬链接的限制：链接在一起的文件必须在同一个文件系统。（df命令，输出的每一行的第一个字段是文件系统名）
* 部分linux只允许root用建立目录的硬链接。

##### 查看当前账户名
```shell
root@zhtjun:~# whoami
root
root@zhtjun:~# id -un
root
```
##### 查看所属组
Linux系统的每个用户至少属于一个组，groups查看当前用户属于哪些组。 查看其他组  groups otherAccount
```shell
root@zhtjun:~# groups
root
#id或者id- a 也可以查看所属组，同时还可查看账户号uid和各个组的组号gid
root@zhtjun:~# id -a
uid=0(root) gid=0(root) groups=0(root)
```
groups输出的第一个组叫首要组，其他为次要组。

##### useradd
useradd可用来建立用户帐号。帐号建好之后，再用passwd设定帐号的密码．而可用userdel删除帐号。使用useradd指令所建立的帐号，
实际上是保存在/etc/passwd文本文件中。
* -c<备注> 　加上备注文字。备注文字会保存在passwd的备注栏位中。
* -d<登入目录> 　指定用户登入时的启始目录。目录 指定用户主目录，如果此目录不存在，则同时使用-m选项，能创建主目录。
* -D 　变更预设值．
* -e<有效期限> 　指定帐号的有效期限。
* -f<缓冲天数> 　指定在密码过期后多少天即关闭该帐号。
* -g<群组> 　指定用户所属的群组。
* -G<群组> 　指定用户所属的附加群组。
* -m 　自动建立用户的登入目录。
* -M 　不要自动建立用户的登入目录。
* -n 　取消建立以用户名称为名的群组．
* -r 　建立系统帐号。
* -s<shell>　 　指定用户登入后所使用的shell。
* -u<uid> 　指定用户ID。
```shell
#添加一般用户
useradd test
#为添加的用户指定相应的用户组
useradd -g root test
#创建一个系统用户
useradd -r test
#为新添加的用户指定home目录
useradd -d /home/mydir test
#建立用户且制定ID
useradd test -u 544
```

**userdel可删除用户帐号与相关的文件。若不加参数，则仅删除用户帐号，而不删除相关文件。**
`userdel [-r][用户帐号]`
* -r 　删除用户登入目录以及目录中所有文件。

##### chmod改变权限命令
chmod命令用来改变文件权限和目录权限，选项为`[ugoa][+-=][rwx-]`,其中u表示文件的所有者（user）,
g表示文件所属组（group）,表示除所有者和所属组组员以外的的其他账户（other）,a表示所有账户（all）,
+,-,=分别表示对权限的增加，减少，定义。

r，w，x，-可以用4，2，1，0来表示。如：0 : \-\-\-,1 : \-\-x,7 : rwx等

##### SUID,SGID及粘滞位

**password**修改密码后，文件/etc/shadow会被更新。
```shell
zhangtejun@zhangtejun-pc:~$ ls -l /etc/shadow
-rw-r----- 1 root shadow 1116 2月  15 10:54 /etc/shadow
```
从表面上看仅root才有权限修改次文件。然而其他账户是如何更新次文件？

```shell
#查看可执行文件password
root@zhtjun:/usr/bin# which passwd 
/usr/bin/passwd
root@zhtjun:/usr/bin# ls -l /usr/bin/passwd 
-rwsr-xr-x 1 root root 51096 May 26  2012 /usr/bin/passwd
```
上面的s就是SUID,SUID是set uid的缩写。SUID仅对可执行二进制文件起作用。

SUID的作用：其他账户在执行时具有文件所有者的权限。chmod u+s sh.sh

简单理解：执行password命令就是执行/usr/bin/passwd，而/usr/bin/passwd有SUID属性并且所有者是root,那么其他账户执行password
时，就临时具有和root账户一样的权限，将加密后的密码写入文件/etc/shadow。

```shell
root@zhtjun:/usr/bin# ls -l |grep '^...s'
-rwsr-sr-x 1 daemon daemon    55456 Oct  3  2014 at
-rwsr-xr-x 1 root   root      46264 May 26  2012 chfn
-rwsr-xr-x 1 root   root      41272 May 26  2012 chsh
-rwsr-xr-x 1 root   root      68024 May 26  2012 gpasswd
-rwsr-xr-x 1 root   root      36432 May 26  2012 newgrp
-rwsr-xr-x 1 root   root      51096 May 26  2012 passwd
-rwsr-sr-x 1 root   mail      89280 Sep  4  2014 procmail
-rwsr-xr-x 2 root   root     113048 Mar  1  2013 sudo
-rwsr-xr-x 2 root   root     113048 Mar  1  2013 sudoedit
-rwsr-sr-x 1 root   root      14264 May  7  2013 X
```

SGID是set gid的缩写。即其他账户执行时具有文件所属组的权限。

设置SGID和SUID属性时需要确保目录或文件对于所属组有执行权限。如果没有会看见大写的S。这是不正常的。需要增加相应权限就正常。

**粘滞位**也就SBIT，是sticky bit的缩写。只有目录才可以设置该属性。可以理解为防删除位。
作用是：在一个大家都具有权限的目录下，某个账号不能随便删除别人的文件或目录。
增加SBIT属性：chmod o+t folder
