# 计划任务

# 单一任务计划

`at`命令可以运行单一任务计划，前提是要开启`atd`服务

atd服务

```shell
# 查看状态
$ systemctl status atd
# 重启服务
$ systemctl restart atd
# 开启服务
$ systemctl start atd
# 停止服务
systemctl stop atd
```

```shell
$ at [-mldv] TIME
$ at -c 任务号码

选项与参数：
-l ：列出所有该使用者的at计划，相当于atq
-d ：取消at计划任务，相当于atrm
-v ：以
-m ：
```





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
