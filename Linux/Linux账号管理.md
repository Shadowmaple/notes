# Linux账号管理

# 用户配置项

一个用户登录shell终端的过程：

1.  查找`/etc/passwd`是否存在该用户账号。没有则退出，若存在则将该账号对应的UID、GID（位于`/etc/group`）以及其家目录与shell设置（如果有的话）读取出来。
2.  核对密码表。利用在`/etc/shadow`中找出的对应账号（UID）的密码，对输入的密码进行核对。
3.  核对通过，进入shell管理阶段。

当要登录LInux主机时，`/etc/passwd`和`/etc/shadow`两个文件就必须要让系统读取。

`/etc/passwd`文件的结构：

```shell
账号名称:密码:UID:GID:用户信息说明栏:家目录:shell

# 示例
$ head -n 5 /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
```

一行代表一个用户，每行的7个字段用 冒号（`:`） 区分。

密码为x，是由于实际的密码已经被迁移到了`/etc/shadow`中。

UID为0，代表的是系统管理员，若想让其它账号也拥有root权限，将其UID设为0即可。一台系统上的系统管理员可以不只是root

除了0为系统管理员，其它的（UID必为正整数）都为一般账号。其中1-999为系统账号，这是为了让系统上的网络服务或后台服务以较小的权限去运行；1000以后为可登陆账号，用于可用于登录的用户使用。

`/etc/shadow`文件的结构：

1.  账号名称
2.  密码
3.  最近修改密码的日期
4.  密码不可被修改的天数
5.  密码需要重新修改的天数
6.  密码需要修改期限前的警告天数
7.  密码失效日
8.  账号失效日期
9.  保留字段

同样的一行一用户，每个字段用冒号分隔，但是有9个字段。

```shell
$ sudo head -n 5 /etc/shadow
root:$6$jnNUj7JA$W9lLmA8opI/4MjQkU31.EJI63ujHZW5qcWpzb3HkPXqdo54i2p4R6RSGMjJueQEetFbF5wU1C5noTsciQYKb/.:18137:0:99999:7:::
daemon:*:18113:0:99999:7:::
bin:*:18113:0:99999:7:::
sys:*:18113:0:99999:7:::
sync:*:18113:0:99999:7:::
```

密码字段内的数据是经过编码的密码（摘要）。由于固定的摘要算法产生的密码是特定的，所以当修改这个字段后，该密码就会失效（无法算出）。最简单的失效方法就是在密码字段钱加上 ! 或 * ，很多程序都是这么做的。

第三栏记录的是最近修改密码的日期，但要注意的是，计算Linux日期的时间是以1970年1月1日作为1而累加的日期，所以它显示的日期会有点”奇怪”。

密码有效日期为【更新日期（第三字段）】+【重新修改日期（第五字段）】，过了该期限后用户仍未修改密码，那么密码就算过期了，这时密码拥有一个过期特性。密码过期特性就是密码过期后，登录系统时系统会强制要求你必须重新设置密码才能登录继续使用。

账号失效就是在超过规定的账号失效日期后，该账号将永久无法使用。这个功能常被用于收费服务中。

# 用户组配置项

用户组的配置文件是`/etc/group`和`/etc/gshadow`。

`/etc/group`文件结构

```shell
组名:用户组密码:GID:用户组支持的账号

$ head -n 5 /etc/group
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:syslog,lawler
```

初始用户组的用户群已经不会加入第四个字段，所以一般来说都是空的（如root，bin等）。

有效用户组：在拥有多个用户组的用户中，当前各种操作（如创建文件）所代表的用户组。

查看有效与支持用户组：

```shell
$ groups
lawler adm cdrom sudo dip plugdev lpadmin sambashare docker
```

所列出的都是当前用户所支持的用户组，但只有第一个才是有效用户组。

切换有效用户组：

```shell
$ newgrp 用户组
```

但要注意的是，`newgrp`命令是通过提供另外一个shell来修改目前用户的有效用户组的，就是说切换后的用户是通过另一个shell的子进程进行登陆的。所以想要切换回来不要再`newgrp`一下，直接`exit`退出子进程shell即可。

`/etc/gshadow`文件结构

1.  组名
2.  密码栏，开头为 ! ，* 或为空时表示无合法密码，所以无用户组管理员
3.  用户组管理员 账号
4.  该用户组支持的账号 

该文件最大的作用就是建立用户组管理员。



# 增删用户

useradd

```shell
$ useradd [-u UID] [-g 初始用户组] [-G 次要用户组] [-mM]\
> [-c 说明栏] [-d 家目录绝对路径] [-s shell] 账号名

选项与参数：
-M : 强制，不要建立使用者家目录，系统账号默认
-m : 强制，建立使用者家目录，一般账号默认
...

# 例子
$ useradd lawler
```

一般情况下，我们根据系统的默认值添加用户即可，也就是直接`useradd 账号`。

系统默认添加用户时会有这几个操作：

+   在`/etc/passwd`中建立一行与账号相关的数据，包括建立UID/GID/家目录。
+   在`/etc/shadow`中将此账号的密码的相关参数写入，但尚未有密码
+   在`/etc/group`中加入一个与账号名称相同的组名
+   在`/home`下建立一个与账号同名的家目录，权限为700

由于在`/etc/shadow`中仅会有密码参数而不会有加密过的密码数据，因此在建立用户账号时还需使用 `passwd 账号` 来设置密码才算是真正完成了用户建立的全部流程。

系统账号（如mysql）主要是用来执行系统所需服务的权限设置，所以系统账号默认不会主动建立家目录。

查看useradd的默认值，源于`/etc/default/useradd`文件：

```shell
$ useradd -D

GROUP=100
HOME=/home
INACTIVE=-1				# 密码过期后是否会失效的设置值
EXPIRE=					# 账号失效日期
SHELL=/bin/sh
SKEL=/etc/skel			# 家目录参考基准目录
CREATE_MAIL_SPOOL=no	# 是否建立用户的mailbox
```

useradd在建立账号时，至少会参考以下文件：

+   `/etc/default/useradd`
+   `/etc/login.defs`

+   `/etc/skel/*`

passwd

在默认情况下，使用 useradd 建立账号之后，该账号还是暂时被锁定的，即无法登录。需要先使用`passwd`设置密码。

``` shell
# 所有人均可使用来修改自己的密码
$ passwd [--stdin] [账号]
# root功能
$ passwd [-luS] [--stdin] 账号

选项与参数：
--stdin : 通过前一个管道的数据作为密码输入
-l : 锁定，密码失效
-u : 解锁，密码恢复
-S : 列出密码相关参数
...

# 示例
$ passwd lawler
```

root功能十分强大，可以设置任意密码。一般账号修改密码时不允许改短，只能改长，但是用root可以改短，只不过会警告。

chage

```shell
$ chage [-ldEImMW] 账号

选项与参数：
-l : 列出账号的详细密码参数
...

# 示例
$ chage -l lawler
```

userdel

删除用户的相关数据：

+   用户账号/密码相关参数：`/etc/passwd`，`/etc/shadow`
+   用户组相关参数：`/etc/group`，`/etc/gshadow`
+   用户个人文件数据：`/home/username`，`/var/spool/mail/username`

```shell
$ userdel [-r] 账号

参数：
-r ： 连同使用者的家目录也一起删除

# 示例
$ userdel -r test
```

如果账号只是**暂时不启用**的话，那么讲`/etc/shadow`里的账号失效日期（第八字段）设为0就可以让该账号无法使用。

# 用户功能

id

用于查询某人或自己的相关UID/GID等信息

```shell
$ id [username]

# 示例
# 查看lawler用户信息
$ id lawler
# 查看当前用户信息
$ id
uid=1000(lawler) gid=1000(lawler) groups=1000(lawler),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lpadmin),126(sambashare),999(docker)
```

chsh

change shell 的简写，用于更改当前用户的默认shell，需要root权限

```shell
$ chsh [-s]

选项与参数：
-s ： 设置修改shell

# 示例
$ chsh -s /bin/zsh
# 也可以不带参数，根据提示修改
$ chsh
```

# 增删用户组

## groupadd

```shell
$ groupadd [-g gid] [-r] 用户组

选项与参数：
-g ： 后接GID，用来直接设置某个GID
-r ： 建立系统用户组，与 /etc/login.defs 内的GID_MIN有关

# 示例
$ groupadd group1
```

## groupmod

用于修改 group 的相关参数

```shell
$ groupmod [-g gid] [-n group_name] 用户组

选项与参数：
-g ： 修改既有的GID数字
-n ： 修改既有的用户组名称

# 示例
# 将group1名称改为test，GID为233
$ groupmod -g 233 -n test group1
```

## groupdel

删除用户组

```shell
$ groupdel 用户组
```

要注意，在删除用户组时，必须要确认 `/etc/passwd` 内的账号没有任何人使用该用户组作为初始用户组，否则无法删除。

## 用户组管理员

用户管理员可以管理该用户组内账号的加入或移除。设置用户管理员使用`gpasswd`。

root用户的操作：

```shell
$ gpasswd [-A user1,...] [-M user2,...] 用户组
$ gpasswd [-rR] 用户组

选项与参数：
   ： 若无任何参数，表示设置该用户组的密码
-A ： 设置用户管理员
-M ： 将账号加入该用户组
-r ： 讲用户组的密码删除
-R ： 讲用户组的密码栏失效

# 示例
# root用户
$ gpasswd -A lawler test
```

用户管理员的功能：

```shell
$ gpasswd [-ad] user 用户组

选项与参数：
-a ： 将用户加入用户组
-d ： 将用户删除用户组

# 示例
$ gpasswd -a nick test 
```



