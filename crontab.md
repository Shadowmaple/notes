# 计划任务

# 单一任务计划

`at`命令可以运行单一任务计划，前提是要开启`atd`服务

atd服务

```shell
# 查看状态
systemctl status atd
# 重启服务
systemctl restart atd
# 开启服务
systemctl start atd
# 停止服务
systemctl stop atd
```

at用法

```shell
$ at [-mldv] TIME
$ at -c 任务号码

选项与参数：
-l ：列出所有该使用者的at计划，相当于atq
-d ：取消at计划任务，相当于atrm
-v ：以较明显的时间格式列出at计划中的任务列表
-c ：可以列出后面接的该项任务的实际命令内容
-m ：当at的任务完成后，即使没有输出信息，亦发email通知使用者该任务已完成
TIME ：时间格式，执行任务的时间，格式有：
  HH:MM  在今日的HH:MM时刻执行，若该时刻已超过，则明天的该时间执行
  HH:MM YYYY-MM-DD
  HH:MM[am|pm] [Month] [Date]	ex> 04pm July 30
  HH:MM[am|pm] + number [minutes|hours|days|weeks]
  	ex> now + 5 minutes		在某个时间后多长时间执行
```

示例：

```shell
$ at now + 3 days
warning: commands will be executed using /bin/sh
at> echo "hello, at" > a.md 
at> <EOT>
job 1 at Sat Sep 14 21:24:00 2019
```

at命令可以打开一个at shell的环境来设置任务命令。

最好用绝对路径来执行你的命令，以避免出问题。

at任务是后台执行的。系统会将at任务独立出你的bash环境，直接交给系统的atd程序来接管。

可以利用batch来使系统有空时才执行后台任务。batch是在CPU的任务负载小于0.8的时候才执行工作任务。任务负载是CPU在单一时间点所负责的任务数量，而非CPU的使用率。

查看任务负载：

```shell
$ uptime
21:41:58 up 13:11,  1 user,  load average: 0.23, 0.39, 0.40
```

其实batch是利用at来执行 ，只是加入了一些控制参数而已。但batch不支持时间参数，可以拿来作为判断是否立刻要执行后台程序的依据。

```shell
$ batch
at> /usr/bin/updatedb
at> <EOT>
job 4 at Sat Sep 14 21:37:00 2019
```

# 循环计划任务


修改默认编辑器：`select-editor`

## 基本指令

cron服务指令

```shell
service cron start
service cron restartw
service cron stop
service cron status
service cron reload
```

crontab操作指令

```shell
$ crontab [-u username] [-l|-e|-r]
选项与参数:
-u ：root身份，操作其它用户的crontab任务
-l ：列出任务内容
-e ：编辑任务
-r ：删除所有任务

# 示例
crontab -l
crontab -e
```

`/etc/cron.deny`：写入的账号禁止使用crontab

当用户使用crontab来建立计划任务之后，该任务就会被记录到`/var/spool/cron`中，且是以账号来作为判断依据的。

命令执行时，最好使用绝对路径

## 系统配置文件

`/etc/crontab`：系统的例行性任务

cron服务的最低检测限制是分钟，所以cron会每分钟去读取一次`/etc/crontab/`和`/var/spool/cron`里面的数据内容

查看`/etc/crontab`文件：

```shell
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
```

和用户的配置文件相比，多了一项用户字段，代表执行命令的用户身份

cron默认有三个地方会执行脚本配置文件：

+   `/etc/crontab`
+   `/etc/cron.d/*`
+   `/var/spool/cron/*`（不同发行版会不相同）

使用总结：

+   个人化操作使用`crontab -e`
+   系统维护管理使用`vi /etc/crontab`
+   自己开发软件使用`vi /etc/cron.d/newfile`

## 配置演示

[crontab parser](https://crontab.guru/#0_*_*_*_*)

# 停机期间任务的唤醒

使用`anacron`，可以将因为某些原因（如停机）导致的超过时间而未执行的任务唤醒，重新执行。

anacron也是每小时被cron执行一次，然后anacron再去检测相关的计划任务是否被执行，若有，则执行，完毕后便停止。

```shell
$ anacron [-sfn] [job]
$ anacron -u [job]

选项与参数：
-s ：开始连续执行各项任务，会根据时间判断是否执行
-f ：强制执行，而不去判断时间记录文件的时间戳
-n ：立刻执行未执行的任务，而不延迟等待时间
-u ：仅更新事件记录文件的时间戳，不执行任何任务
```

```shell
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
HOME=/root
LOGNAME=root

# 天数 延迟时间 工作名称 执行的命令串
1   5   cron.daily  run-parts --report /etc/cron.daily
7   10  cron.weekly run-parts --report /etc/cron.weekly
@monthly    15  cron.monthly    run-parts --report /etc/cron.monthly
```

延迟时间是为了防止和其它资源的冲突，单位为分钟