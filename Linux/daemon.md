# 系统服务

+   `/lib/systemd/system/`：各个服务最主要的启动脚本设置
+   `/etc/systemd/system`：管理员根据主机系统的需求所建立的执行脚本，优先级比`/lib/systemd/system/`高

操作系统启动到底会不会执行某些服务其实是看`/etc/systemd/system`下面的设置，所以该目录下是一堆链接文件。而实际执行的 systemd 启动脚本配置文件，其实都是放置在`/lib/systemd/system/`下面，因此若想要修改某个服务启动的设置，就要去`/lib/systemd/system/`下面修改才行。

systemd 的unit 类型分类

| 扩展名     | 主要服务功能                                                 |
| ---------- | ------------------------------------------------------------ |
| .service   | 一般服务类型，主要是系统服务，包括服务器本身需要的本地服务和网络服务等 |
| .socket    | 内部程序数据交换的socket服务，主要是IPC（Inter-process communication）的传输信息socket文件功能。 |
| .target    | 执行环境类型，其实就是一群unit的集合 |
| .mout<br>.automount | 文件系统挂载相关的服务，如来自网络的自动挂载、NFS文件系统挂载与文件系统相关性较高的进程管理 |
| .path | 检测特定文件或目录类型 |
| .timer | 循环执行的服务，由System主动提供的crontab功能 |

基本上，Systemd这个启动服务的机制，主要是通过 systemctl 命令来完成。和以前的`System V`需要 service、chkconfig、setup、init 等命令来协助不同，Systemd 只有 systemctl 这一个命令来处理。

```shell
$ systemctl [command] [unit]

command选项：
start		：启动
stop		：停止
restart		：重启
status		：查看状态
reload		：不关闭unit的情况下，重新加载配置文件，让设置生效
enable		：设置开机启动
disable		：设置开机不启动
is-active	：是否在运行中
is-enabled	：是否开机启动
mask		：强制注销，非删除
unmask		：注销恢复

# 例
$ systemctl status docker.service
```

不应该使用 kill 的方式来关闭一个正常的服务，否则 systemctl 会无法继续监控该服务。

daemon 常见状态：

+   active（running）：正在运行
+   active（exited）：仅执行一次就正常结束的服务，目前并没有任何进程在系统中执行。
+   active（waiting）：正在运行当中，但还需等待其它的事件发生才能继续运行。
+   inactive（dead）：未运行

daemon 的默认（启动）状态：

+   enabled：开机启动
+   disabled：开机不启动
+   static：不可自己启动运行，但会被其它的 enabled 的服务来唤醒，是依赖属性的服务
+   mask：无论如何都无法被启动，已被强制注销（非删除）

查看系统所有服务：

```shell
$ systemctl [command] [--type=TYPE] [--all]

command选项：
	list-units		：依据 unit 显示目前有启动的unit，加上--all会列出全部
	list-unit-files	：列出文件列表说明，只有 unit file 和 state 两栏
	
# 例子
$ systemctl
$ systemctl list-units --type=service --all
# 使用list-unit-files看着更加舒服
$ systemctl list-unit-files
```

操作环境（target unit）：

+   multi-user.target：纯命令行模式

+   graphical.target：命令加上图形界面，包含 multi-user.target
+   rescue.target：额外的临时系统（安全模式）
+   emergency.target：紧急处理系统的错误
+   shutdown.target：关机模式
+   getty.target：设置tty的配置文件

+   ....

```shell
# 获取目前的target
$ systemctl get-default

# 获取目前target的依赖
$ systemctl list-dependencies
# 获取multi-user.target 的依赖
$ systemctl list-dependencies multi-user.target

# 获取当前target被使用的target
$ systemctl list-dependencies --reverse
# 获取multi-user.target被使用的target
$ systemctl list-dependencies multi-user.target --reverse
```

# service 类型的配置文件

配置文件的三个主要部分：

+   [Unit]：unit本身的说明，一级与其它依赖 daemon 的设置，包括在什么服务之后才会启动等等
+   [Service]、[Socket]、[Timer]、[Mount]、[Path]：不同 unit 类型的不同设置项目。主要用来规范服务启动的脚本、环境配置文件、重新启动的方式等
+   [Install]：将此unit放入哪个target中去

