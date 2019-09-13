# su与sudo

su 和 sudo 是两个用于用户身份切换的命令，主要是一般用户与root之间的切换。

用户身份的切换是开启另一个shell，作为子进程运行的。所以要恢复原先的用户用exit即可。

# su

su 是最简单的身份切换命令，它可以进行任何身份的切换

```shell
$ su [-lm] [-c 命令] [username]

选项与参数：
-  ：使用-则代表使用login-shell的变量文件读取方式来登录系统。
-l ：与-相同
-m ：表示使用目前的环境配置，而不读取新使用者的配置文件
-c ：仅进行一次命令
```

后面不加 username 则默认切换为 root

单纯的使用 `su` 切换为root的身份，读取的变量设置方式为非登录shell的方式，不会修改大部分原有的变量。所以要用root账户，最好还是使用 `su -`

`su -c`：切换为root执行一次命令，且执行完毕后就恢复原本的身份

```shell
# 切换为root执行一次命令，且执行完毕后就恢复原本的身份
$ su -c ls

# 切换为其它用户
$ su - nick
$ su -l nick
```

用法总结：

+   若要完整地切换到新用户的环境，必须要使用`su - username`或`su -l username`
+   使用`su - -c command`来执行一次root命令
+   su的切换需要新用户的密码，但使用root切换到其它用户时，不需输入新用户的密码



# sudo

sudo 的切换仅需要自己的密码。sudo可以让当前用户以其它用户的身份执行命令。

仅有规范到`/etc/sudoers`内的用户才能够执行sudo命令

```shell
$ sudo [-b] [-u 新的用户]

选项与参数：
-b ：命令后台执行
-u ：接切换的用户
```

sudo的执行流程：

1.  系统于`/etc/sudoers`文件中查找该用户是否拥有sudo的执行权限
2.  若具有该执行权限，便让用户输入自己的密码以确认
3.  密码输入成功，开始进行sudo后续接的命令

root执行sudo时，不需要输入密码

能否使用sudo关键在于`/etc/sudoers`文件，但最好不要直接用vi等方式去编辑，因为该文件的内容有一定的规范，所以需要用`visudo`去修改这个文件。

`/etc/sudoers`中的数据格式：

```shell
使用者账号	登录者的来源主机名=（可切换的身份） 可执行的命令
```

登录者的来源主机名：该账号是由哪一台网络主机连接过来的。该设置值可以指定客户端计算机（信任的来源）

可切换的命令：务必要用绝对路径

```shell
$ sudo visudo
...
# 用户
root	ALL=(ALL:ALL) ALL
# 用户组
%admin	ALL=(ALL) ALL
# 免密码执行
test1 	ALL=(root) NOPASSWD: ALL
# 只能使用passwd命令
%test2	ALL=(root) /usr/bin/passwd
...
```

有 % 号的是用户组

免密码执行要加 `NOPASSWD`关键词

sudo 是有有限时间的，在有效时间内密码无需再次输入，这个有效时间是5分钟。

