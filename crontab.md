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



# crontab


修改默认编辑器：`select-editor`

# 基本指令

```shell
crontab -l
crontab -e
crontab -r
service cron start
service cron restartw
service cron stop
service cron status
service cron reload
```

# 代码演示
[crontab parser](https://crontab.guru/#0_*_*_*_*)
