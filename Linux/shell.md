# shell

## 硬件、内核与shell
硬件：
整个系统中的实际工作者，包括磁盘、显卡、网卡、CPU、内存等

内核：
真正控制硬件工作的东西，含有CPU调度、内存管理、磁盘输入输出等工作

使用者界面（shell、KDE、应用程序）：
接受来自使用者的命令，以与内核进行沟通

命令和图形界面：
用户操作操作系统

我们通过shell将输入的命令与内核沟通，好让内核可以控制硬件来正确无误地工作

## shell
查看当前可用的shell
```shell
$ cat /etc/shells
```
系统上合法的shell要写入`/etc/shells`这个文件中

当我们顺利在终端（tty）上登录后，linux就会根据`/etc/passwd`文件的设置给我们一个默认的shell

查看登录的用户取得的shell
```shell
$ cat /etc/passwd
```
结果
```shell
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
......
lawler:x:1000:1000:Lawler,,,:/home/lawler:/bin/zsh
cups-pk-helper:x:121:116:user for cups-pk-helper service,,,:/home/cups-pk-helper:/usr/sbin/nologin
redis:x:122:127::/var/lib/redis:/usr/sbin/nologin
mysql:x:123:128:MySQL Server,,,:/nonexistent:/bin/false
```

alias

type：查询命令是否为bash的内置命令
```shell
$ type cd
cd is a shell builtin   ==>shell的内置命令
$ type ls
ls is an alias for ls --color=tty
$ type vi
vi is /usr/bin/vi
```
type主要找出可执行文件，所以也可以用来作为类似which命令的用途

|     组合键      |              功能               |
| :-------------: | :-----------------------------: |
| ctrl+u / ctrl+k |   从光标处向前/向后删除命令串   |
| ctrl+a / ctrl+e | 移动光标到命令串的最前面/最后面 |
## 变量
echo -> 读取变量
```shell
$ echo $SHELL
/bin/zsh
$ echo ${HOME}
/home/lawler
```
变量的设置

+ "=" 连接，两边不允许空格
+ 双引号内的特殊字符(如`$`），可以保留原本的特性
+ 单引号内的特殊字符则仅为一般字符（纯文本）
+ 可用转义符【\】将特殊符号（`$`、\、空格、回车等）变成一般字符
```shell
$ myname=Shadow
$ echo $myname
Shadow
$ c="myname is $myname"
$ echo $c
myname is Shadow
```
export -> 设置或转为环境变量
declare --> 将环境变量转为自定义变量

unset -> 取消变量

反单引号【`】：在其内的命令会被先执行
```shell
$ ls -ld `locate crontab` #先以locate将文件名数据列出，再以ls处理以获取各个文件的权限
$ ls -ld $(locate crontab) #效果同上
```

查看环境变量：export 或 env

用 set 观察所有变量（包括环境变量和自定义变量）

PS1：提示字符的设置
```shell
➜  ~ git:(master) ✗ echo $PS1
PS1='${ret_status} %{$fg[cyan]%}%c%{$reset_color%} $(git_prompt_info)'
```

语系变量：locale

查看支持的语系
```shell
$ locale -a
```

### 变量键盘读取
```shell
read [-pt] variable
```
```shell
$ read -t 5 na  #5秒内从键盘读取变量na
333
$ echo $na
333
```
### 变量类型声明 —— declare 或 typeset
```shell
declare [-aixr] variable
-a -> 定义为数组类型
-i -> 定义为整型
-x -> 变为环境变量
-r -> 设置为readonly类型，即变量不可更改，也不能unset
```
+ 变量类型默认为字符串
+ bash环境中的数值运算，默认最多仅能达到整数形态，所以1/3的结果为0
+ 对于只读类型，通常要注销再登录才能恢复该变量的类型

### 与文件系统及程序的限制关系 —— ulimit
ulimit：限制用户的某些系统资源，包括可以开启的文件数量，可使用的CPU时间，可使用的内存总量等