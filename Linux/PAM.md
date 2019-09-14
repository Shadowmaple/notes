# PAM 机制验证流程

PAM（Pluggable Authentication Modules），插入式验证模块，是一套验证的机制。

PAM相当于一套API，提供一连串的验证机制。用户将验证的需求告知PAM，PAM则返回验证的结果。

PAM机制实现了“高内聚，低耦合”的设计理念。

PAM用来进行验证的数据称为模块。



PAM调用流程

以执行`passwd`命令为例，该程序调用PAM机制的流程如下：

1.  执行`/usr/bin/passwd`程序，并输入密码
2.  passwd程序调用PAM模块进行验证
3.  PAM模块会到`/etc/pam.d/`（或其它`*/etc/pam.d/`）中寻找与改程序（passwd）同名的PAM配置文件
4.  根据该配置文件内的设置和规则，引用相关的PAM模块进行验证
5.  将验证结果返回给该程序
6.  该程序根据PAM的返回结果决定之后的操作



PAM配置文件的格式：

```shell
验证类别 控指标准 PAM模块 模块参数
```

本机ubuntu18.04的PAM配置文件在`/snap/core18/1144/etc/pam.d`中，一般的Linux系统是在`/etc/pam.d`中

```shell
$ cat /snap/core18/1144/etc/pam.d/su

auth       sufficient pam_rootok.so
session    required   pam_env.so readenv=1
session    required   pam_env.so readenv=1 envfile=/etc/default/locale
session    optional   pam_mail.so nopen
session    required   pam_limits.so
@include common-auth
@include common-account
@include common-session
```



验证类别（Type）

验证类别主要有4种：

+   auth：验证类接口，主要用来检验用户的身份
+   account：账户管理接口，执行与访问、帐户及凭证有效期、密码限制/规则等授权相关的操作
+   session：会话类接口，用于记录用户登录与注销时的信息
+   password：密码管理接口，主要执行密码修改等操作



验证的控制标识（control flag）

控制标识就是验证通过的标准，用于管控该验证的控制方式：

+   required：若验证失败则带有”失败“的标志，即本次的验证一定失败，但无论成功或失败，本次的验证流程都不会被打断，只有整个栈运行完毕才会返回”失败“的信号。
+   requisite：若验证失败则立即结束验证流程并返回”验证失败“的信号，若成功则继续后续的流程。
+   sufficient：若验证成功则立即结束验证流程，并返回”success“的信号，若失败则继续后续的流程。
+   optional：大多用于显示信息方面，不用于验证
+   include：调用其它配置文件来作为此类别的验证
+   substack：运行其他配置文件中的流程，并将整个运行结果作为该行的结果进行输出

`substack`和 `include` 的不同点在于认证结果的作用域：

如果某个流程栈 `include` 了一个带 `requisite` 的栈，这个 `requisite` 失败将直接导致认证失败，同时退出栈；而某个流程栈 `substack` 了同样的栈时，`requisite` 的失败只会导致这个子栈返回失败信号，母栈并不会在此退出。



PAM控制标识的返回流程：



PAM验证机制流程

以login为例：

1.  验证阶段：

