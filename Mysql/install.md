# Mysql安装

> [官方安装教程](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/)

从官方下载 Mysql APT Repository 的 Deb Package ，以便 apt 可以获取到最新的 mysql 软件。

> https://dev.mysql.com/downloads/repo/apt/



```shell
$ sudo apt-get update

$ sudo apt-get install mysql-server
```

在安装的过程中会要求设置root密码，选择身份验证方法，身份验证方法不选新的，而要选第二个旧的5.0验证方法，因为ubuntu暂时还不支持8.0的验证加密方式

启动

```shell
$ sudo service mysql status
$ sudo service mysql stop
$ sudo service mysql start
```

可以选择安装其它版本，会启动一个对话框让你选择

```shell
$ sudo dpkg-reconfigure mysql-apt-config
$ sudo apt-get update
```

再重新安装



## 配置

```shell
$ sudo mysql_secure_installation
```

```shell
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No: N（选择N，不会进行密码的强校验）
Please set the password for root here.

New password: 

Re-enter new password: 
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : N（选择N，不删除匿名用户）

 ... skipping.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : N（选择N，允许root远程连接）

 ... skipping.
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : N（选择N，不删除test数据库）

 ... skipping.
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y（选择Y，修改权限立即生效）
Success.

All done! 
```





## 连接数据库

```shell
$ sudo mysql -uroot -p

# 或者
$ mysql -h 127.0.0.1 -P 3306 -uroot -p123456
# -h 为远程IP，-P 为端口号，-u 为用户名，-p 为密码
```





## 将字符编码设置为UTF-8

查看字符编码

```mysql
> show variables like '%char%';
```


打开mysql配置文件`sudo vim /etc/mysql/my.cnf`

```shell
 # 在[client]下追加：
default-character-set=utf8

# 在[mysqld]下追加：
character-set-server=utf8

# 在[mysql]下追加：
default-character-set=utf8
```

修改后，重启MySQL服务器

