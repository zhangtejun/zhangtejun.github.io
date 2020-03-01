---
layout: post
title:  "linuxShell"
date:   2017-07-08 10:32:00
author: zhangtejun
categories: linux
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

`ls -i或者--inode`查看文件的inode。每一个文件或者目录对应一个inode。
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

# 设置SUID要保证文件同时拥有所有者的可执行（x）权限，否则会出现一个大写的S,这是不正常的
-rwSr-xr-x 1 root root 51096 May 26  2012 a.sh
```
上面的s就是SUID,SUID是set uid的缩写。SUID仅对可执行二进制文件起作用。

SUID的作用：其他账户在执行时具有文件所有者的权限。chmod u+s sh.sh / chmod u+4777 sh.sh

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

SGID是set gid的缩写。即其他账户执行时具有文件所属组的权限。chmod g+s sh.sh / chmod 2777 folder

设置SGID和SUID属性时需要确保目录或文件对于所属组有执行权限。如果没有会看见大写的S。这是不正常的。需要增加相应权限就正常。

**粘滞位**也就SBIT，是sticky bit的缩写。只有目录才可以设置该属性。可以理解为防删除位。
作用是：在一个大家都具有权限的目录下，某个账号不能随便删除别人的文件或目录。
增加SBIT属性：chmod o+t folder/chmod +t folder  / chmod 1777 folder

##### alias,unalias
设置别名：`alias 别名=值`

列出当前所有别名：`alias`或者`alias -p`

##### 修改.bashrc
以Ubuntu系统为例，/etc/environment,/etc/profile,/etc/bash.bashrc三个文件控制着整个系统的环境设置。而每个用户主目录下的.bashrc
的修改只影响用户自己的环境。

##### source命令和点命令
内置source命令和点命令都可以使修改马上生效。

#### 命令的解释顺序和改变解释顺序的3个内置命令
Bash解释顺序：alias-->keyword-->function-->built in-->$PATH
即别名，关键字，函数，内置命令，外部命令。

改变解释顺序的3个内置命令
* commoand <命令> 忽略函数和别名，按内置和外部命令来处理。
* builtin<命令> 只查找内置命令。
* enable 禁止或使能内置命令。enable -n禁止，不带参数使能

##### 标准错误输出
执行一个shell命令会自动打开标准错误输出文件（stderr）,默认对应终端屏幕，对应文件描述符为2。
```shell
root@zhtjun:~# ls -l test.txt
ls: cannot access test.txt: No such file or directory
#No such file or directory 不是标准输出而是标准错误

#使用2>对标准错误输出重定向，2>>追加重定向。注意2和>间不能有空格。

#对2类输出信息分别重定向到不同文件标准输出到a.txt,标准错误输出到b.txt。1>中的1可以省略。
ls -l test.txt 1>a.txt 2>b.txt
#2>&1将标准错误输出重定向到标准输出
ls -l test.txt 1>a.txt 2>&1
#&>和>&可以将标准错误和标准输出重定向到同一个文件。通常情况下标准错误输出到设备文件/dev/null
ls -l test.txt &>a.txt
```
##### 同时把结果输出到标准输出和文件 tee
文件描述符0，1，2分别对应标准输入，标准输出，标准错误。
```shell
root@zhtjun:~/weblogic# ls -l test.txt | tee info.txt
-rw-r--r-- 1 root root 0 Jul  8 22:31 test.txt
#如果info.txt有内容将被覆盖。追加内容为：tee -a/--append info.txt 
#tee - 文件用‘-’代替将输出到屏幕
root@zhtjun:~/weblogic# cat info.txt 
-rw-r--r-- 1 root root 0 Jul  8 22:31 test.txt
```
常用重定向命令

|    命令格式             |        含义           |
| ------------- |:-------------:|
| command > file     | 标准输出重定向到文件 |
| command 1> file      | 同上      | 
| command \>\> file | 标准输出追加重定向到文件     |
| command 2> file | 标准错误重定向到文件     |
| command > file 2> file2 | 标准输出重定向到文件file,标准错误重定向到文件file2     |
| command > file 2>&1 | 标准输出和标准错误同时重定向到文件     |
| command &> file | 同上     |
| command >& file | 同上     |
| command \>\> file 2>&1| 标准输出和标准错误同时追加重定向到文件     |
| command &\>\> file| 标准输出和标准错误同时追加重定向到文件 ,bash4支持  |
| command < file| 命令以文件作为标准输入     |
| command < file > file2| 以file作为命令的输入，标准输出重定向到file2     |
| command \|tee file| 命令的结果显示在终端，结果同时写入文件file     |
| command \|& command2| command的标准输出和标准错误以管道传给command2,bash4支持，等价于  command 2>&1 \| command2   |

##### 命令后台执行 &
```shell
cp -r dir1 dir2 &
[1] 6090
```
第一个是工作号也叫作业号1，6090为进程号。

windows下文件的换行符是`\r\n`，linux下换行符是`\n`。

##### 大括号和小括号中的命令
一条命令或多条命令可以放在大括号或小括号里执行。在小括号时，命令在一个子shell执行。大括号在当前shell里执行。
```shell
root@zhtjun:~# pwd
/root
root@zhtjun:~# (cd /tmp)
root@zhtjun:~# 
```

##### 子shell
子shell和父shell有各自的进程号，
```shell
root@zhtjun:~# pwd;ps
/root
  PID TTY          TIME CMD
14522 pts/0    00:00:00 bash #当前shell的进程id 14522
15225 pts/0    00:00:00 ps   #命令ps的进程id 15225
root@zhtjun:~# (pwd;ps)
/root
  PID TTY          TIME CMD
14522 pts/0    00:00:00 bash #当前父shell的进程id，为改变
15237 pts/0    00:00:00 bash #当前子shell的进程id
15238 pts/0    00:00:00 ps
root@zhtjun:~# 
```
#### 变量和数组

##### 变量定义
bash无需声明其变量的类型就可以直接赋值。引用变量时需要加美元符号$。定义变量时必须是**字母数字和下划线组和**。

内置命令unset可以清楚变量。`unset 变量名`
```shell
zhangtejun@zhangtejun-pc:~$ testa=100
zhangtejun@zhangtejun-pc:~$ echo $testa 
100
zhangtejun@zhangtejun-pc:~$ testb="hello"
zhangtejun@zhangtejun-pc:~$ echo $testb
hello
```
定义字符串时如果包含空格需要使用引号。*双引号中可以包含变量，而单引号将原封不动输出所有内容。*
```shell
zhangtejun@zhangtejun-pc:~$ a=hello world
bash: world: 未找到命令
zhangtejun@zhangtejun-pc:~$ a='hello world'
zhangtejun@zhangtejun-pc:~$ a="hello world"
zhangtejun@zhangtejun-pc:~$ a=$A_world     #将变量A_world赋值给a
zhangtejun@zhangtejun-pc:~$ a=${A}_world   #将变量A和_world拼接后赋值给a
```
大括号除了可作为变量的定界符，还有扩展功能：
```shell
root@zhtjun:~# echo a{d,c,b{x,y,zz}}e
root@zhtjun:~# ade ace abxe abye abzze
root@zhtjun:~# mkdir -p /tmp/mail/{a,b,c} #将分别创建abc目录。
```
##### 命令执行结果赋值变量
* **变量=\`命令\`**(注意是反引号)
* 变量=$(命令)

```shell
zhangtejun@zhangtejun-pc:~$ A=`date`
zhangtejun@zhangtejun-pc:~$ echo $A
2017年 07月 09日 星期日 12:12:17 CST
zhangtejun@zhangtejun-pc:~$ B=$(date)
zhangtejun@zhangtejun-pc:~$ echo $B
2017年 07月 09日 星期日 12:12:29 CST
```


##### Bash的特殊变量

|    变量             |        含义           |
| ------ |:----------------------------------:|
| $0      | 当前脚本的文件名 |
| $N      | 传递给脚本或函数的参数$1,$2...${10}。N大于9时需要用括号括起来 |
| $#      | 传递给脚本或函数的参数个数 |
| $*      | 传递给脚本或函数的所有参数。(一起作为一个字符串) |
| $@      | 传递给脚本或函数的所有参数。(分别作为一个字符串) |
| $?      | 上个命令的退出状态值 |
| $$      | 当前脚本的文件名 |
| $$      | 当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。|
| $!      | 最后一个后台命令的进程ID|
| $-      | 当前shell的选项 |
| $_      | 上一条命令的最后一个参数 |


##### 字符串操作

|    表达式             |        含义           |
| ------------- |:-------------:|
| ${#str}      | 字符串str的长度 |
| ${str:position}      | 从字符串str中位置position开始提取子字符串 |
| ${str:position:length}      | 从字符串str中位置position开始提取长度为length的子字符串|
| ${str#regexp}      | 从变量str的开头，删除最短匹配regexp的子串 |
| ${str##regexp}     | 从变量str的开头，删除最长匹配regexp的子串 |
| ${str%regexp}      | 从变量str的结尾，删除最短匹配regexp的子串 |
| ${str%%regexp}     | 从变量str的结尾，删除最长匹配regexp的子串 |
| ${str/regexp/replacement}      | 使用replacement，来替换第一个匹配的regexp |
| ${str//regexp/replacement}      | 使用replacement，来替换所有匹配的regexp |
| ${str/#regexp/replacement}      | 如果str的前缀匹配regexp， 使用replacement，来替换所有的regexp |
| ${str/%regexp/replacement}      | 如果str的后缀匹配regexp， 使用replacement，来替换所有的regexp|

##### expr处理字符串

|    命令             |        含义           |
| ------------- |:-------------:|
| expr length str      | 字符串str的长度 |
| expr index str  char    | 计算字符char在字符串str中首次出现的位置,没有找到返回0 |
| expr substr  pos str      | 从字符串str中位置pos开始提取长度为length的子字符串|
| expr match str   regexp   | 字符串str开头的匹配regexp的长度 |
| expr str :regexp      | 字符串str开头的匹配regexp的长度 |
| expr match str   :\\(regexp\\)     | 从字符串str开头位置提取regexp |
| expr str   :\\(regexp\\)     | 从字符串str开头位置提取regexp |

expr也可以计算表达式
格式为： expr 表达式

#### 正则表达式

##### 多字符串替换*
```shell
root@zhtjun:~/weblogic# ls
a_copy.txt  a_link.txt  info.txt  test.txt
# 查看当前目录下所有文件
root@zhtjun:~/weblogic# cat *
test
222
```
##### 单字符串？
```shell
root@zhtjun:~/weblogic# ls -lh
total 12K
-rw-r--r-- 1 root root    0 Jul 15 10:44 a
-rw-r--r-- 1 root root    5 Jul  8 10:58 a_copy.txt
-rw-r--r-- 1 root root    4 Jul  8 11:06 a_link.txt
drwxr-xr-x 2 root root 4.0K Jul 15 10:44 b
-rw-r--r-- 1 root root    0 Jul  8 22:31 info.txt
-rw-r--r-- 1 root root    0 Jul  8 22:31 test.txt
# ?代表一个字符，ls ? 将显示名字为一个字符的文件或文件夹
# ls ?(n个) 将显示名字为n个字符的文件或文件夹
# ls x?.* 将显示名字为匹配该参数的文件或文件夹
root@zhtjun:~/weblogic# ls ?
a

b:
```
##### 范围替换[]与[!]
单字符串的替换除了用？之外，还可以用中括号。
```shell
# [amz]*表示名字以a,或m，或者以z开头的文件名。
ls [amz]*
# 匹配名字为小写字母的单字符文件名，ls [a-z][a-z]匹配双字符
ls [a-z]

# [!az]*表示不以字符a或z开头的文件名。[!a-z]*不以字符a到z开头的文件名
ls [!az]*
```

##### 过滤器grep
命令grep(global search regular expression and print out the line )

|    字符             |        含义           |
|:-------------:|:-------------|
|.      | 匹配如何单个字符串 |
| *    |匹配0次或多次 |
| ?    | 匹配0次或1次(扩展) |
| +   |匹配1次或多次(扩展) |
| \\{N\\}  |精确匹配N次(扩展的用{N}) |
| \\{N,\\}  |匹配N次或多次(扩展的用{N,}) |
| \\{N,M\\}  |匹配最少N次,至多M次(扩展的用{N,M}) |
| a\|b\|c  |匹配a或b或c (扩展)|
| ( )  |分组符号，如read(a\|b)匹配reada或者readb(扩展)|
| [x-y]  |范围，如[0-9]匹配数字|
| [] |匹配一组字符中的一个，[abc8]表示匹配a，b，c，8中的任意一个|
| [^] |列表范围取反，[^abc8]表示匹配a，b，c，8之外的任意字符|
| ^    |匹配行始 |
| $    |匹配行末 |
| \    |转义 |
| \b    |单词定界符 |
| \\<    |词首定界符 |
| \\>    |词尾定界符 |
| \w    |匹配字母数字下划线（单词字符）即[A-Za-z0-9_] |
| \W    |\w的取反 |

```shell
#grep . 将过滤掉空行，显示所有非空行
root@zhtjun:~# grep . test.txt

#过滤出包含abc的行,-i或者\-\-ignore-case来忽略大小写。
root@zhtjun:~# grep abc  test.txt

#过滤出已book结尾的行
root@zhtjun:~# grep book$  test.txt

#过滤出空行，$匹配行末，^匹配行始，则^$ 匹配空行
root@zhtjun:~# grep ^$  test.txt

#过滤出非空行，可以用-v,用来取反invert
root@zhtjun:~# grep -v ^$  test.txt

#匹配包含数字的行
root@zhtjun:~# grep [0-9]  test.txt

#匹配包含2个连续字母o的行
root@zhtjun:~# grep 'o\{2\}'  test.txt

#匹配以t开头，e结尾，中间有2个字符的单词的行。任意字符为：'\<t*e\>'
root@zhtjun:~# grep '\<t..e\>'  test.txt

#匹配包含2个连续字母o的行,-n用来显示匹配的行号，-c或者--count用来匹配次数(行数)，
root@zhtjun:~# grep -nc 'o\{2\}'  test.txt
```
正则表达式中的‘^’和-v都有取反功能，但是具体作用不太一样。

##### 扩展的egrep
egrep是grep的扩展，支持多元字符。
```shell
#[0-9]?表示0个或者1个阿拉伯数字，在字符串abc2ef中包含一个数字，可以用[0-9]?匹配，
root@zhtjun:~# echo abc2ef | egrep '[0-9]?'
abc2ef

#grep 无法过滤出
root@zhtjun:~# echo abc2ef | grep '[0-9]?'
root@zhtjun:~#

# 'egrep' means 'grep -E'.  'fgrep' means 'grep -F'.
root@zhtjun:~# echo abcd2ef | grep -E '[0-9]?'
abcd2ef
root@zhtjun:~# 

#
root@zhtjun:~# echo "it readable" | egrep 'read(able|ability)'
it readable

root@zhtjun:~# echo "it readable" | grep 'read(able|ability)'
root@zhtjun:~# 


```

##### 剪取内容命令cut

```shell
#显示每一行的第一个字符,显示前3个字符为：-c1-3可以简写为-c-3
root@zhtjun:~# cut -c1 linux.txt

#显示每一行的第一个字符和第3到第5个字符
root@zhtjun:~# cut -c1,3-5 linux.txt

#显示每一行的第一个字符和第3到第5个字符和第10个到行尾的字符
root@zhtjun:~# cut -c1,3-5,10- linux.txt

#文件内容为 aaa bbb ccc ，-f用来指定域（field list）,第一个和第3个域 -f1,3。默认分隔符为<tab>键（\\t）
root@zhtjun:~# cut -f2 linux.txt
bbb

#分隔符不是<tab>，可以用-d来指定分隔符
root@zhtjun:~# cut -d: -f1,7 /etc/passwd
root:/bin/bash
daemon:/bin/sh
bin:/bin/sh
sys:/bin/sh
sync:/bin/sync
games:/bin/sh

```

##### 合并相应行命令paste
paste和cut作用相反

```shell
#默认分隔符为<tab>
root@zhtjun:~# paste linux.txt linux1.txt linux2.txt 
aaa bbb ccc

#-d指定分隔符
root@zhtjun:~# paste -d# linux.txt linux1.txt linux2.txt 
aaa#bbb#ccc

#默认分隔符为<tab>，将所有账户名一行显示
root@zhtjun:~# cat /etc/passwd | cut -d: -f1 | paste -s
root    daemon  bin     sys     sync    games   man     lp      mail    news    uucp    proxy   www-data        backup  list    irc     gnats   nobody  libuuid Debian-exim     statd   sshd    ubuntu  messagebus saned   weblogic        mysql

#指定分隔符为空格
root@zhtjun:~# cat /etc/passwd | cut -d: -f1 | paste -d' ' -s
root daemon bin sys sync games man lp mail news uucp proxy www-data backup list irc gnats nobody libuuid Debian-exim statd sshd ubuntu messagebus saned weblogic mysql
```
##### 转换或删除命令tr

```shell
#将linux.txt中的所有字符i替换为x。将小写替换为大小 tr [a-z] [A-Z]<Linux.txt 
root@zhtjun:~# tr i x<Linux.txt

#大小写转换还可以用[:upper:]和[:lower:]
root@zhtjun:~# cat linux.txt | tr [:upper:] [:lower:]

#-s用来压缩重复字符

#-d用来删除字符， 如：去空格 tr -d' '

```

##### 排序sort

```shell
#默认升序排列，-v/--reverse来倒序排序
root@zhtjun:~# cat /etc/passwd | cut -d: -f1 |sort

#-u 去掉重复行
root@zhtjun:~# cat /etc/passwd | cut -d: -f1 |sort -u

#-R 随机排序
root@zhtjun:~# cat /etc/passwd | cut -d: -f1 |sort -R

#对应数值文件内容排序，-n按照数值大小排序
root@zhtjun:~# cat /etc/passwd | cut -d: -f1 |sort -n

#按指定域排序,数值域
root@zhtjun:~# sort -k 2 -n linux.txt

#按指定域排序,非数值域，第3列，默认分隔符为<tab>
root@zhtjun:~# sort -k 3  linux.txt

#-o指定输出存放的文件名，可以和当前文件重名
root@zhtjun:~# sort   linux.txt  -o linux.txt 
```


##### sed 流编辑器
```shell
#替换命令s/regexp/replacement/，把linux.txt中的linux替换为LINUX。
#替换命令s/regexp/replacement/flags。flags为g表示全局替换，为数值3时代表替换第3个匹配，gi忽略大小替换
root@zhtjun:~# sed 's/Linux/LINUX/g'linux.txt


#替换命令s/regexp/replacement/，把linux.txt中的linux替换为LINUX
root@zhtjun:~# sed 's/Linux/LINUX/'linux.txt


#每行前2个替换为A
root@zhtjun:~# sed 's/^../A/'linux.txt

#删除每行前2个字符
root@zhtjun:~# sed 's/^..//'linux.txt

#将文件每行的第一个空格到行尾替换为B,注意：/ .间有个空格
root@zhtjun:~# sed 's/ .*$/B/'linux.txt

#注释掉5到7行
root@zhtjun:~# sed '5,7s/^/#/'linux.txt

#使用-r/--regexp-extended时，sed支持扩展正则表达式
root@zhtjun:~# echo 'morning#afternoon'>qq #存入文件qq
#.+代表一个或多个字符，\1和\2分别代表第1个括号和第二个括号里的匹配。\t代表<tab>键。
#如果需要保存至文件，可以使用重定向或者-i选项，sed -i -r 's/(.+)#(.+)/\2\t\1/' qq 
#sed -i_bak -r 's/(.+)#(.+)/\2\t\1/' qq ,备份文件为qq_bak为原内容。
root@zhtjun:~# sed -r 's/(.+)#(.+)/\2\t\1/' qq 
morning		afternoon'

#原封不动的输出内容,相当于cat Linux.txt
root@zhtjun:~# sed '' Linux.txt

#原封不动的输出内容,相当于cat Linux.txt,-n的作用是取消默认输出
root@zhtjun:~# sed '' Linux.txt

#显示文件前2行,没有-n的话，前2行将输出2次，一次为命令p的输出，一次是默认输出。
root@zhtjun:~# sed -n '1,2p' Linux.txt

#显示文件第一行,显示文件第三行到尾行，sed -n '3,$p' Linux.txt
root@zhtjun:~# sed -n '1p' Linux.txt

#显示文件中包含Linux的行，
root@zhtjun:~# sed -n '/Linux/p' Linux.txt

#删除文件前2行，
root@zhtjun:~# sed -n '1,2d' Linux.txt

#删除包含linux的行，
root@zhtjun:~# sed -n '/Linux/d' Linux.txt


#将文件中包含Linux的行，存入L2.txt
root@zhtjun:~#  cat Linux.txt | sed -n '/Linux/w' L2.txt

#将文件中包含Linux的行前，插入内容Good
root@zhtjun:~#  cat Linux.txt | sed -n '/Linux/i\Good' L2.txt


```

|    命令             |        含义           |
|:-------------:|:-------------|
|a\\     | 在当前行之后添加一行或者多行，多行时除最后一行外，每行末尾需要需加续行符\\ |
|c\\     |用新文本替换当前行中的文本，多行时除最后一行外，每行末尾需要需加续行符\\    |
|d|删除文本|
|i\\     |在当前行之前插入文本，多行时除最后一行外，每行末尾需要需加续行符\\  |
|l   |显示不可打印字符    |
|p|打印文本|
|r|从文件中读取输入行|
|s|匹配查找或替换|
|w|将所选文本写入文件|

##### 一行多命令和保存匹配&

```shell
#将Linux的Jack替换为Rose,并且将包含interest的行删除
root@zhtjun:~#  sed -e 's/Jack/Rose/' -e '/interest/d' Linux.txt
root@zhtjun:~#  sed 's/Jack/Rose/;/interest/d' Linux.txt
#s/[a-z]^u&/g小写转大写，将[a-z]将的字符，\u的作用是转大写，&保存已匹配的字符（串）以便调用它
root@zhtjun:~#  sed 's/Jack/Rose/;/interest/d' Linux.txt;s/[a-z]/\u&/g' Linux.txt 

#保存匹配&
#将Jack替换为Hello
root@zhtjun:~# echo Jack | sed 's/Jack/Hello/'
Hello
#在Jack前加Hello（而不用Hello替换），因为&代表已匹配上的Jack
root@zhtjun:~# echo Jack | sed 's/Jack/Hello &/'
Hello Jack
```

##### sed的退出状态
grep命令在找到了它所查找的模式时，退出状态为0，若没有找到，推出状态非0。

sed和awk都支持查找模式,但是无论是否查询到，退出状态都为0，当语法错误时，退出状态为非0.

sed可以说是一条交互式的编辑命令，另一方面sed也是一门脚本语言。格式为：sed [选项] -f sed-script 文件名
,其中sed-script是事先已写好的sed脚本。在一行中可以有多条sed命令，当过长时，sed脚本更方便。
```
root@zhtjun:~# cat sed.sh
#!bin/sed -f
s/Jack/Rose/
/interest/d
#在最后一行的前面加上Yours sincerely。$代表最后一行
$i\Yours sincerely
#在最后一行的后面加上Yours sincerely。$代表最后一行
$a\In BeiJing
```

##### AWK文本处理工具
awk的名称来自它的三个创始人，Alfred Aho,Peter Weinberger,Brian Kernighan的姓氏首字母。

格式：awk 'command' fileName

awk对文件进行逐行扫描并处理。

```shell
#$0代表awk读入的文件的每一行内容。 相当于cat test.txt
root@zhtjun:~# awk '{print $0}' test.txt

#输出加上序号。awk '{print NR, $1 ,$2}' test.txt    $1代表第1列。 
# NR（Number of the current Record）为awk内置变量，表示当前记录（当前行）的序号
root@zhtjun:~# awk '{print NR, $0}' test.txt 
1 11 22 33
2 44 55
3 66
4 
5 77 88 99

# 依次读取文件每一行并输出Good。 test文件共5行
root@zhtjun:~# awk '{print "Good"}' test.txt 
Good
Good
Good
Good
Good
```


awk支持2种不同的变量，内建变量和自定义变量
* 内建变量（预定义变量）
  1. $n 当前记录的第n个字段，比如n为2标识第二个字段
  2. $0 这个变量包含执行过程中当前行的文本内容
  3. FILENAME 当前输入的文件名
  4. FS 字段分隔符（默认是空格）
  5. NF 表示字段数，在执行过程中对应当前字段数，NF：列的个数
  6. FNR 各文件分别计数的行号
  7. NR 表示记录数，相当于行号
  8. OFS 输出字段分隔符 默认空格
  9. ORS 输出记录分隔符 默认换行符
  10. RS 记录分隔符 默认换行符

* 常用命令选项
  1. -F 指定分隔符
  2. -v 赋值一个用户自定义变量
  3. -f 指定脚本文件，从脚本中读取awk命令


```shell
# 输出 BB
echo "AA BB CC" | awk '{print $2}' # 默认空格分隔符
echo "AA BB CC" | awk -F " " '{print $2}' # 指定空格分隔符 -F和分隔符之间可以有空格
echo "AA BB CC" | awk -F" " '{print $2}' # 指定空格分隔符

echo "AA,BB,CC" | awk -F,   '{print $2}' # 指定逗号分隔符
echo "AA,BB,CC" | awk -F ,   '{print $2}' # 指定逗号分隔符
echo "AA,BB,CC" | awk -F","   '{print $2}' # 指定逗号分隔符
echo "AA,BB,CC" | awk -F ","   '{print $2}' # 指定逗号分隔符

# 输出c3
echo "A1b2c3" | awk -F "b2"   '{print $2}' # 指定b2分隔符
echo "A1b2c3" | awk  'BEGIN {FS="b2"} {print $2}'# 使用FS指定b2分隔符，FS的值 必须是双引号
```

**关系运算符**
```shell
echo "1 2 3 " > a.txt
awk '{print $1+10}' a.txt  # => 11

# 取最后一列
awk '{print $NF}' a.txt  # => 3
awk '{print $3}' a.txt  # => 3

# 取倒数第二列
awk '{print $(NF-1)}' a.txt  # => 2


```

##### awk支持正则表达式
格式： awk '查找模式 [命令]' 文件名

```shell
#表示第二列匹配88的行
root@zhtjun:~# awk '$2~88' test.txt
77 88 99

#取出第1 2 列另存为test.bak文件
root@zhtjun:~# awk '{print $1 ,$2 >"test.bak"}' test.txt
```

另一种常用格式： awk [-F 域分隔符]'命令' 文件名

默认分隔符为空格，则可以省略-F。
```shell

root@zhtjun:~# awk -F: '{print $1 ,$7}' /etc/passwd
# 实际上-F后面跟得分隔符指的是输入域分隔符，也可以由awk内置变量FS指定；相应的输出分隔符由awk的内置变量OFS指定，默认都是空格。
root@zhtjun:~# awk '{FS=":";OFS="#"; print $1 ,$7}' /etc/passwd
	
```
awk 有BEGIN和END模块，简单来说，BENGIN总是先执行，END总是后执行。
```shell
root@zhtjun:~# awk 'BEGIN{print "列1 列2 列3 列4"}{print $0}' test.txt
列1 列2 列3 列4
11 22 33
44 55
66

77 88 99

#awk具有流程控制类似C语言，如if,while,for等。还有内置的数学函数。字符串处理行数。调用系统行数。
#awk还可以写成脚本，格式为： awk [选项] -f awk-script[文件名]
root@zhtjun:~#  awk '{if($2>87){print $0}}' test.txt
#打印一行后就退出
root@zhtjun:~#  awk '{if($2>87){print $0；exit}}' test.txt
```

##### 进程和作业
正在运行的一个或者多个进程称为一个作业。
命令ps查看正在系统中运行的进程信息。
```SHELL
# 分别是进程标识好PID(Process ID)，终端号 TTY，进程占用CPU时间，相应的命令。
root@zhtjun:~# ps
  PID TTY          TIME CMD
 2627 pts/0    00:00:00 bash
 2680 pts/0    00:00:00 ps

# 加上选项-f，查看显示多信息。（f是full-format listing的意思），包括父进程标识号PPID，进程启动时间STIME和命令参数。 
root@zhtjun:~# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root      2627  2625  0 09:19 pts/0    00:00:00 -bash
root      2890  2627  0 09:22 pts/0    00:00:00 ps -f
```

##### 挂进进程<Ctrl+Z>
挂起进程后，可以用内置命令jobs查看作业状态。作业号只在当前窗口有效，进程则是在整个系统中有效。
```shell
# 1是作业号，Stopped是组欧俄的一种状态，表示暂停不是完全停止。
root@zhtjun:~# jobs
[1]+  Stopped                 vi test.txt

# 加上-l可查看进程ID，选项-p,仅显示进程ID。选项-s,显示状态为 Stopped的作业。选项-r,显示状态为运行中（Running）状态的作业。
root@zhtjun:~# jobs -l
[1]+  3276 Stopped                 vi test.txt
root@zhtjun:~# 
```
##### 前台fg和后台fg
```shell
# +代表最近作业，-代表前一个作业
root@zhtjun:~# jobs
[1]-  Stopped                 vi test.txt
[2]+  Stopped                 cat
# 当前2条后台命令，希望吧莫条命令拿到前台（foreground）,可以用内置命令fg，格式为fg [作业号]
```

|    引用            |        所值的后台作业           |
|:-------------:|:-------------|
|%N    | 编号为N的后台作业 |
|%string   | 命令以string开头的作业 |
|%?string  | 命令包含string的作业 |
|%+   | 最近被放到后台的作业 |
|%%    | 同上 |
|%-    | 第二近近被放到后台的作业 |

命令bg把作业到后台执行。
##### 发送信号命令kill
kill -l可以列出所有信号名和编号

用命令kill给作业或者进程传递信号的格式为：kill [-s 信号名 | -n 信号编号 | -信号名 | -信号编号] 作业号或者进程ID，
如果没有指定信号名，默认信号是SIFGTERM。
```shell
root@zhtjun:~# find / -name "*.gz" 2>/dev/null
/home/apache/httpd-2.2.31.tar.gz
/home/nodeproject/microblog.tar.gz
^Z
[1]+  Stopped                 find / -name "*.gz" 2> /dev/null
# 运行jobs，可见当前有一个作业，状态为stopped。
root@zhtjun:~# jobs
[1]+  Stopped                 find / -name "*.gz" 2> /dev/null
# 如果想终止该作业，用kill %1
root@zhtjun:~# kill %1

[1]+  Stopped                 find / -name "*.gz" 2> /dev/null
# 再次查看，作业状态变为Terminated（因为收到默认信号TERM）
root@zhtjun:~# jobs
[1]+  Terminated              find / -name "*.gz" 2> /dev/null
# 由于各种原因用默认信号的kill命令有时会终止不了进程，信号KILL，即编号为9的信号可以来强行终止进程或作业。
# 用KILL强行终止,进程ID也可以换成作用号。还可以换成kill -KILL 123321或者kill -n 9 123321 或者 kill -p 123321
root@zhtjun:~# kill -s SIGKILL 123321(进程ID)

```

##### 等待命令wait
内置命令wait的格式为：wait [id]
id是进程ID或者作业号。

##### 捕获信号<Ctr+c>
Ctr+c可以用来中断正在运行的程序，如果按下Ctr+c时不仅希望程序停止运行，还希望做一些其他操作，可以通过内置命令trap实现。
格式为：trap commands signals 

commands代表一组命令，用分号隔开。signals代表一组信号，用空格隔开。trap的作用是为signals设置陷阱，当shell收到signals中的某个信号时，
就运行commands。

命令trap -l和kill -l的效果是一样的。

信号SIGKILL和SIGSTOP不能被捕获，阻塞或忽略，就是说对这2个信号设置陷阱是无效的，命令kill -19(或者-STOP，或者-SIGSTOP)加进程号或作业ID可强行暂停进程或作业。

终止不掉的进程可以用kill -9(或者kill -KILL 或者kill -SIGKILL)来强行终止，是因为信号KILL不能被捕获。

##### 移除作业的命令disown
格式为：disown [-h] [-ar] [作业号]
命令disown加作业号不是用来停止某个作业的，而是从当前的作业表中移除该作用，移除后jobs就查找不到了，ps仍然可以查找到。
-r的作用是移除当前shell处于running状态的作业。
-h 是使作业忽略SIGHUP信号
-ar 使所有的作业忽略SIGHUP信号

##### 暂停shell的命令suspend
