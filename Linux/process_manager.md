# 进程管理

# 进程

程序（program）：一个二进制文件，放置在存储媒介（如硬盘、光盘、软盘、磁带等）中，以物理文件的形式存在。

进程（process）：运行中的程序。程序被触发后，执行者的**权限与属性**、程序的代码与所需数据等都会被加载到内存中，操作系统给予这个内存中的单元一个 PID。

进程拥有执行者的权限，由这个进程衍生出来的其它子进程在一般状态下，也会沿用这个进程的相关权限。

子进程拥有PID和PPID（Parent PID）。

遇到的问题：有时将一个进程关闭后，发现过一阵子它又产生了，并且其PID还不一样。这是因为它是子进程，只要有父进程存在，那么当杀掉其子进程后，父进程就会主动再产生一个。所以要擒贼先擒王，杀掉那个父进程。

fork-and-exec：程序调用流程

Linux 的程序调用通常为 fork-and-exec 流程。

进程都会借由父进程以复制（fork）的方式产生一个一模一样的临时进程（但具有PPID且PID不一样），然后被复制出来的临时进程再以 exec 的方式来执行实际要执行的进程，最终称为一个子进程。

服务（daemon）：常驻在内存中的进程。在后台启动并一直持续不断地运行。

一般 daemon 类型的进程都会在文件名后加上 d，以便简单地判断该进程是否为daemon，如 crond，atd

每个进程可能是独立的，亦可能有依赖性，只要到独立的进程中，删除有问题的那个进程，他就会被系统删除掉。

所以 Linux 几乎不会存在宕机的问题。因为它可以在任何时候，将某个被困住的进程杀掉，然后再重新执行该进程而不用重新启动。若在 Linux 以命令行界面登录，在屏幕中显示错误信息后就挂了，动都不能动，怎么办？这个时候就可以动用系统默认的七个终端界面（tty），随意切换到任意一个终端界面，用 `ps -aux`找出错误的进程，然后 `kill`一下就ok了。

>   Linux系统默认提供六个命令行界面，和一个图形界面

# 任务管理

该任务管理（job control）是用在 bash 环境下的，即当我们登录系统获取 bash shell 之后，在单一终端下同时执行多个任务的操作管理。

执行任务管理的操作中，其实每个任务都是目前 bash 的子进程，即彼此之间是由相关性的，我们无法用任务管理的方式由 tty1 的环境去管理 tty2 的bash。

执行 bash shell 的任务管理的限制：

+   shell 只管理自己的shell 所触发的任务的进程
+   前台：可以控制与执行命令的环境
+   后台：可以自动执行的任务，无法使用 ctrl+c 来终止它
+   后台中执行的进程不能等待 terminal 或 shell 的输入

后台任务的状态有暂停挂起（suspended）和运行（running）

>   ubuntu的暂停是suspended，而在另外一些的发行版中的标记为stop

&：后台执行命令

```shell
$ typora Linux/process_manager.md&
[1] 13731
# [job number] PID
```

执行后在终端界面打印出任务号码和PID。job number 只与该shell环境有关。

巧妙运行数据流重定向：`2>&1 &`

ctrl+z：将目前的任务丢到后台中并暂停

jobs：查看目前后台任务的状态

```shell
$ jobs [-lrs]

选项与参数：
-l ：列出job number与命令串的同时，列出PID
-r ：仅列出正在后台run的任务
-s ：仅列出正在后台stop的任务

# 例子
$ jobs -l
[1]    13731 running    typora Linux/process_manager.md
[2]  - 14879 suspended  vim concept.md
[3]  + 15001 suspended  vim goland.md

$ jobs %typora
[1]  - running    typora Linux/process_manager.md
```

job number 后的 + 代表当前默认的使用任务（最近被放入的任务），而 - 代表次要（第二个）默认任务。这个标识的意义在于，当输入 fg 时，带有 + 号的任务会被拿到前台来处理。而最近的第三个及以后的命令不在会有+或-的标识。

fg：将后台任务拿回前台处理

```shell
# 例子
# 取出默认的那个+的任务，默认为最近任务
$ fg
# 取出规定任务号码的任务
$ fg %1
```

bg：将任务在后台下的状态变为running

```shell
# 将当前默认的任务，变为running
$ bg
[1]  + 13731 continued  typora Linux/process_manager.md

# 将任务号码为1的任务变为running
$ bg %1
```

kill：管理后台任务

```shell
$ kill -signal %jobnumber
$ kill -l

选项与参数：
-l :列出目前kill能够使用的信号（signal）
signal：给予后接任务的指令
	-1：重新读取一次参数的配置文件
	-2：代表执行与ctrl+c同等的操作
	-9：立即强制删除一个任务
	-15：以正常的进程方式终止一项任务，与-9不同，默认删除方式
```

kill 后接的数字默认会被当做PID，要管理任务的话，必须要加%号

nohup：脱机管理任务

```shell
# 前台任务
$ nohup [command] 
# 后台任务
$ nohup [command] &

# 例
$ nohup ./run.sh &
```

值得注意的是，nohup并不支持bash内置的命令，所以命令必须是外部命令才行

nohup 与终端无关，其信息的输出默认会被定向至`nohup.out`文件

# 进程管理

查看进程

ps：静态查看某个时间点的进程

```shell
$ ps

选项与参数：
-e/-A ：显示所有进程
-a ：不显示与终端有关的进程
-f ：列出完整的信息
-u ：有效使用者（effective user）相关的进程
-l ：较长较详细地将PID的信息列出

# 常用例子
# 查看自己shell的进程
$ ps
# 查看所有系统运行的进程
$ ps -ef
$ ps -aux
# 查看当前用户的相关进程
$ ps -u
# 连同部分进程树状态
$ ps -axjf
```

僵尸（zombie）进程：该进程已经执行完毕或即将终止，但该进程的父进程无法完整地将该进程结束，造成该进程一直存在内存当中。一般来说，若某个进程的CMD后带上了`<defunct>`时，就代表该进程是僵尸进程。系统不稳定容易等造成僵尸进程。

通常僵尸进程都已经无法管理，而直接交给systemd这个进程来负责，但它是系统第一个执行的进程，是所有进程的父进程，杀掉它，系统就死了。所以若产生僵尸进程，而系统过一阵子还没有办法通过内核非经常性的特殊处理来将该进程删除时，那就只好用关机重启的方式来kill了。

top：动态查看

```shell
$ top

选项与参数：
-d ：接秒数，表示进程界面刷新的时间，默认3秒
-p ：指定某些个PID来执行查看监测

top界面可用命令：
?：显示top中可输入的按键命令
P：以CPU的使用排序显示
M：以Memory的使用排序显示
N：以PID排序
T：CPU时间的累积（TIME+）排序
k：给予某个PID一个信号
r：给予某个PID重新制定一个nice值
c：改变CMD的显示状态，完整的命令或是缩略命令
m：改变内存使用的显示状态
q：退出

# 5秒刷新
$ top -d 5
```

pstree：显示进程树

```shell
$ pstree

选项与参数：
-p：同时列出进程的PID
-u：同时列出进程所属的账号名称
```

signal

| 代号 | 名称    | 内容                                                      |
| ---- | ------- | --------------------------------------------------------- |
| 1    | SIGHUP  | 重新启动                                                  |
| 2    | SIGINT  | 相当于Ctrl+c来中断进程                                    |
| 9    | SIGKILL | 强制中断一个进程，若执行到一半可能会有半成品的产生，如vim |
| 15   | SIGTERM | 以正常方式结束进程                                        |
| 19   | SIGSTOP | 相当于Ctrl+z来暂停进程                                    |

kill：根据PID删除某个进程
killall：根据进程名称删除某个进程（或是一类服务）

```shell
$ killall -signal

选项与参数：
-i ：删除时出现提示字符
-e ：后接command
-I ：命令名忽略大小写

$ killall -i -9 bash
```



/proc/*

进程都是在内存当中，而内存当中的数据又都是写入到`/proc/*`该目录下

当前主机上的各个进程的PID都以目录的形式存在于 `/proc`当中

```shell
$ sudo ls -l /proc/1
total 0
dr-xr-xr-x 2 root root 0 10月  3 10:50 attr
-rw-r--r-- 1 root root 0 10月  3 10:56 autogroup
-r-------- 1 root root 0 10月  3 10:56 auxv
-r--r--r-- 1 root root 0 10月  3 10:50 cgroup
--w------- 1 root root 0 10月  3 10:56 clear_refs
-r--r--r-- 1 root root 0 10月  3 10:50 cmdline
-rw-r--r-- 1 root root 0 10月  3 10:50 comm
-rw-r--r-- 1 root root 0 10月  3 10:56 coredump_filter
-r--r--r-- 1 root root 0 10月  3 10:56 cpuset
lrwxrwxrwx 1 root root 0 10月  3 10:56 cwd -> /
-r-------- 1 root root 0 10月  3 10:50 environ
lrwxrwxrwx 1 root root 0 10月  3 10:50 exe -> /lib/systemd/systemd
dr-x------ 2 root root 0 10月  3 10:50 fd
.....(省略)
```

其中：

+   cmdline：该进程被启动的命令串
+   environ：该进程的环境变量



fuser：借由文件或文件系统找出正在使用该文件的进程

```shell
$ fuser [-umv] [-ki] [-signal] file/dir

选项与参数：
-u ：同时列出进程拥有者
-m ：主动提到文件系统的最顶层
-v ：列出每个文件、进程与命令的完整相关性
-k ：找出使用该文件/目录的PID，并师徒以SIGKILL给予该PID
-i ：与-k配合，删除前询问

# 例子
$ fuser -uv .
                     USER        PID ACCESS COMMAND
/home/lawler/repository/notes:
                     lawler     4382 ..c.. (lawler)zsh
                     lawler    13731 ..c.. (lawler)Typora
                     lawler    13775 ..c.. (lawler)Typora
                     lawler    13787 ..c.. (lawler)Typora
```

ACCESS项目：

+   c：该进程在当前目录下（非子目录）
+   e：可被触发为执行状态
+   f：一个被开启的文件
+   r：顶层目录
+   F：该文件被使用，但在等待响应中
+   m：可能为共享的动态数据库



pidof：找出正在执行的进程的PID

```shell
$ pidof zsh
 4382
```

