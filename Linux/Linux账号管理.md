# Linux账号管理

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



