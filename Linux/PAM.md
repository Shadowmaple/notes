# PAM 机制验证流程

# 认识

PAM（Pluggable Authentication Modules），插入式验证模块，是一套验证的机制。

PAM相当于一套API，提供一连串的验证机制。用户将验证的需求告知PAM，PAM则返回验证的结果。

PAM机制实现了“高内聚，低耦合”的设计理念。

PAM用来进行验证的数据称为模块。



# PAM调用流程

以执行`passwd`命令为例，该程序调用PAM机制的流程如下：

1.  执行`/usr/bin/passwd`程序，并输入密码
2.  passwd程序调用PAM模块进行验证
3.  PAM模块会到`/etc/pam.d/`（或其它`*/etc/pam.d/`）中寻找与改程序（passwd）同名的PAM配置文件
4.  根据该配置文件内的设置和规则，引用相关的PAM模块进行验证
5.  将验证结果返回给该程序
6.  该程序根据PAM的返回结果决定之后的操作

# 配置文件

PAM 的配置文件可以是 `/etc/pam.conf` 这一个文件，也可以是 `/etc/pam.d/` 文件夹内的多个文件。如果 `/etc/pam.d/` 这个文件夹存在，Linux-PAM 将自动忽略 `/etc/pam.conf`。

PAM配置文件的格式：

```shell
验证类别 控指标准 PAM模块 模块参数
```

查看`su`程序的配置文件

```shell
$ car /etc/pam.d/su
auth       sufficient pam_rootok.so
session       required   pam_env.so readenv=1
session       required   pam_env.so readenv=1 envfile=/etc/default/locale
session    optional   pam_mail.so nopen
session    required   pam_limits.so
@include common-auth
@include common-account
@include common-session
```

# 验证类别

验证类别（Type）主要有4种：

+   auth：验证类接口，主要用来检验用户的身份
+   account：账户管理接口，执行与访问、帐户及凭证有效期、密码限制/规则等授权相关的操作
+   session：会话类接口，用于记录用户登录与注销时的信息
+   password：密码管理接口，主要执行密码修改等操作

# 控制标识

验证的控制标识（control flag）就是验证通过的标准，用于管控该验证的控制方式：

+   required：若验证失败则带有”失败“的标志，即本次的验证一定失败，但无论成功或失败，本次的验证流程都不会被打断，只有整个栈运行完毕才会返回”失败“的信号。
+   requisite：若验证失败则立即结束验证流程并返回”验证失败“的信号，若成功则继续后续的流程。
+   sufficient：若验证成功则立即结束验证流程，并返回”success“的信号，若失败则继续后续的流程。
+   optional：大多用于显示信息方面，不用于验证



PAM控制标识的返回流程：



# 常用的PAM模块

| PAM模块| 说明      |
| ---- | -------- |
| pam_unix.so| 验证阶段的认证功能，授权阶段的账号许可证管理，密码更新阶段检验输入的新密码，会话阶段的日志文件记录 |
| pam_nologin.so | 限制一般用户是否可以登录主机；当 /etc/nologin这个文件存在时，则所有一般用户都无法再登录系统 |
| pam_loginuid.so | 验证用户的UID数值 |
| pam_env.so | 设置环境变量 |
| pam_shells.so | 限制用户登录的shell，必须是包含在/etc/shells文件中 |
| pam_deny.so | 用于拒绝访问 |
| pam_permit.so| 任何时候都返回成功 |
| pam_securetty.so | 限制root用户登录的终端，如果用户要以root登录时，则登录的tty必须在/etc/securetty之中 |
| pam_pwquality.so | 检验密码的强度，提供密码是否在字典中、密码输入几次失败就断开连接等功能|
| pam_limits.so    | 定义使用系统资源的上限，root用户也会受此限制，可以通过/etc/security/limits.conf来设定 |



# PAM验证机制流程

以login为例：

1.  验证阶段：
    1.  先经过`pam_securetty.so`的判断，若用户是root，则会参考`/etc/securetty`的设置
    2.  经过`pam_env.so`设置额外的环境变量
    3.  通过`pam_unix.so`检验密码，若通过则返回 login 程序
    4.  若不通过则以`pam_succeed_if.so`判断UID是否大于1000，若小于1000则返回失败，否则以`pam_deny.so`拒绝连接
2.  授权阶段：
    1.  先以`pam_nologin.so`判断`/etc/nologin`是否存在，若存在则不许一般用户登录
    2.  以`pam_unix.so`和`pam_localuser.so`进行账号管理
    3.  再以`pam_succeed_if.so`判断UID是否小于1000，若小于则不记录登录信息
    4.  最后以`pam_permit.so`允许该账号登录
3.  密码阶段：
    1.  先以`pam_pwquality.so`设置密码仅能尝试错误3次
    2.  接下来以`pam_unix.so`通过`sha512`、`shadow`等功能进行密码检验，若通过则返回程序，若不通过则以`pam_deny.so`拒绝登录
4.  会话阶段：
    1.  先以`pam_selinux.so`暂时关闭SELinux
    2.  使用`pam_limits.so`设置好用户能够操作的系统资源
    3.  登录成功后开始记录相关信息到日志文件中
    4.  以`pam_loginuid.so`规范不同的UID权限
    5.  开启`pam_selinux.so`的功能

# 参考

+   https://www.jianshu.com/p/760f239be916
+   https://www.ibm.com/developerworks/cn/linux/l-cn-pam/index.html
+   https://www.ibm.com/developerworks/cn/linux/l-pam/index.html