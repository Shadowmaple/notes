---
title: Linux的文件属性与权限
date: 2019-05-10 00:14:29
updated:
categories: Linux
tags:
  - linux
description: 对于Linux文件权限部分的所做笔记与许些总结
---
# 权限身份
三种身份：
1. 用户 User
2. 群组 Group
3. 其他人 Others

用户身份和用户组记录的文件：

+ /etc/passwd ： 存储系统账户、一般用户和root账户的相关信息（默认情况下）
+ /etc/shadow ： 记录个人密码
+ /etc/group ： 记录所有组名

# 文件属性
```
➜  HomeWorks 21:15:00 git:(master) ls -l
total 132
-rw-rw-r-- 1 lawler lawler 26041 4月  17 21:19 apiDoc.yaml
-rw-rw-r-- 1 lawler lawler  9355 4月  17 21:20 api.md
drwxrwxr-x 5 lawler lawler  4096 5月   4 14:58 app
-rw-r--r-- 1 lawler lawler 32768 4月  23 09:48 app.db
-rw-rw-r-- 1 lawler lawler 16384 4月  25 22:42 celerybeat-schedule
-rw-rw-r-- 1 lawler lawler   201 4月  16 15:22 celeryout.file
-rw-rw-r-- 1 lawler lawler   194 4月  16 08:08 celery_run.py
-rw-rw-r-- 1 lawler lawler  1199 4月  23 09:49 config.py
-rw-rw-r-- 1 lawler lawler    97 5月   4 23:44 manage.py
-rw-rw-r-- 1 lawler lawler   244 4月  16 15:19 myout.file
drwxrwxr-x 2 lawler lawler  4096 4月  23 09:49 __pycache__
-rw-rw-r-- 1 lawler lawler    55 3月  17 00:14 README.md
-rw-rw-r-- 1 lawler lawler   192 4月  17 21:48 requirement.txt
-rw-rw-r-- 1 lawler lawler   116 3月  30 17:36 run.sh
-rw-rw-r-- 1 lawler lawler   873 3月  17 12:00 test.py
```
第一栏：文件的类型和权限
第一个字符表示文件的类型，permssion
d是目录，【-】是文件，【l】是链接文件，【b】为设备文件里的可供存储的周边设备，【c】为设备文件里面的串行端口设备，如键盘鼠标（一次性读取设备）
接下来的字符
rwx：可读可写可执行（read, write, execute）
分为三组，分别是文件拥有者的权限，该用户组的权限，其他人的权限

第二栏：链接节点数，incode
第三栏：拥有者账号
第四栏：所属用户组
第五栏：容量大小，默认为 Bytes
第六栏：最近的修改时间。格式为日期（月/日）和时间，但若时间过久远，便会仅显示日期和年份。使用`ls -l --full-time`显示完整的时间格式
第七栏：文件名

# 权限意义
| 组件 | ｒ | w | x |
| :---: | :---: | :---: | :---: |
| 文件 | 读取文件内容 | 修改文件内容 | 执行文件 |
| 目录 | 读到文件名，读取目录结构列表 | 修改文件，包括增删文件、更名、移动 | 进入目录 |
注：
1. 在 Linux 下，[x] 权限决定文件能否被执行，这和 Windows 有所不同
2. 文件有 [w] 权限只能对文件内容进行修改，不包含该文件的删除
3. 若目录无 [r] 权限，则无法 list ，在某些系统和shell上甚至无法使用 [tab] 进行自动补全
4. 从网络上下载一个文件时，其属性和权限会发生改变

# 修改文件属性和权限
chgrp： 修改文件所属用户组
chown： 修改文件拥有者以及用户组
chmod： 修改文件权限

## chgrp
```
# 格式
$ chgrp [option] grouup filename

# -R 为递归修改
$ chgrp -R lawler blog/
```
## chown
```
# 格式
$ chown [option] user filename
$ chown [option] user:group filename    # 也可用小数点[.]来间隔

# 例子
# chown lawler:lawler test.c
```
## chmod
数字类型权限表示：`rwx` --> 421
符号类型权限表示：

+ `+ - =` --> 加入，移除，设置
+ `u g o a` --> user，group，others，all

```
# 格式
$ chmod [option] mode filename

# 例子
# 数字类型修改
$ chmod 654 test.sh     # 更改后为 -rw-r-xr--

# 符号类型修改
$ chmod u=rwx,go=rx test.sh     # 设置user为读写执行，用户组和其他为读执行
$ chmod -R a+x blog/       # 都加上可执行权限（该目录递归添加）
```

# 默认权限

在一个文件或目录建立的时候，所使用的是系统提供的默认权限。umask 就是指定用户的默认权限值。

```shell
# 一般权限显示的是数字
$ umask
002

# 符号类型显示
$ umask -S
u=rwx,g=rwx,o=rx
```

但要注意，umask 显示的权限值所代表的是该默认值需要减去的权限。用户建立一个文件的默认最大权限为666，而目录默认最大权限为777，则各自减去默认权限值则分别为：`-rw-rw-r--`，`drwxrwxr-x`。

# 特殊权限

## SUID

Set UiD，简称为SUID，是出现在用户上的一个特殊权限（s），代表在执行特定命令中，不具有该文件所有权的该用户可以暂时获得权限。

SUID的限制与功能：

+ SUID权限仅对二进制程序有效
+ 执行者对于改程序需要有可执行权限
+ 本权限仅在执行该程序的过程中有效
+ 执行者将具有改程序拥有者的权限

例如二进制可执行程序 passwd（`/usr/bin/passwd`）具有s权限，所以可以修改root用户的`/etc/shadow`，但cat（`/bin/cat`）不具有SUID权限，所以不能读取`/etc/shadow`。

```shell
$ ll /usr/bin/passwd
-rwsr-xr-x 1 root root 59K 1月  25  2018 /usr/bin/passwd

$ ll /bin/cat
-rwxr-xr-x 1 root root 35K 1月  18  2018 /bin/cat
```

另外要注意，SUID仅对二进制（可执行）程序有效，对shell脚本和目录无效。

## SGID

Set GID，简称为SGID，是出现在用户组上的一个特殊权限（s），它与SUID类似，但可以针对二进制可执行文件和目录设置。

对文件的限制功能：

+ SGID对二进制程序有效
+ 程序执行者对于改程序需具备x权限
+ 执行者在执行过程中能够短暂获得程序用户组的支持

对目录的限制与功能：

+ 用户若对于此目录具有r和x的权限时，该用户可进入此目录
+ 用户在此目录下的有效用户组将会变成该目录的用户组
+ 若用户在此目录下具有w的权限（可新建文件），则用户所建立的新文件所属的用户组与此用户组相同

在目录下的功能，就是我将短暂具有该目录用户组的身份，并在此新建的东西都会变成这个用户组的，而不是我本身所属的用户组。

## SBIT

Sticky Bit，简称为SBIT，是一个在其他权限组上的一个特殊权限（t），它只针对目录有效，它表示

SBIT的限制与功能：

+ 当用户对于该目录具有w和x权限，即具有写入的权限
+ 当用户在该目录下建立文件或目录时，仅有自己和root才有权利删除该文件

就是说我在这个目录建立了一个文件，我和root可以对它进行删除、更名、移动等一系列操作，但是他人不可删除我这个文件，我也不可以删除他人建立的文件

## 权限设置

设置SUID、SGID、SBIT权限的方式也是数字形式，但是原有的三位数字权限将会变为四位数字权限，第一位为特殊权限，其中SUID为4，SGID为2，SBIT为1。例如 4664 则为具有SUID权限。

用chmod设置即可，数字法设置的区别只是多了一位代表特殊权限的数字而已。也可以通过符号法来设置，SUID为 u+s，SGID为 g+s，SBIT为 o+t，这是加，减去则为-，大体相同。

但是当user、group、others不既有x的可执行权限时，设置了特设权限，这个特殊权限会变成大写（S和T），代表这个权限是空的。
