# Linux 的文件查找

# 脚本文件的查找

## whcih

which 用于查找可执行文件，它是根据PATH的环境变量所规定的路径来查找文件



```shell
# 用法
$ whcih [-a] command

# 选项与参数
-a 	: 列出所有的结果

# 查找which的文件名
$ which -a which
which: shell built-in command
/usr/bin/which
/bin/which
```





# 文件的查找

在linux中，有三个命令可用于文件查找：whereis, locate, find，三者有一定的区别，各有其优劣。

## whereis

whereis 只是查找系统中某些特定目录下的文件，如 /bin/sbin 下面的执行文件和 /usrshare/man 下面的 `man page` 文件，所以查找的速度非常快。但它只是查找几个特定目录下的文件，查找范围十分有限。

可以利用 `whereis -l` 来查看 whereis 究竟查找了多少目录



whereis 的用法

```shell
$ whereis [-bmsu] <filename>

# 选项与参数
-b	: 只查找二进制文件
-m	: 只找在说明文件manuals路径下的文件
-s	: 只找源文件
-u	: 只找不在上述三个项目中的其他特殊文件
```



## locate

locate 是从数据库（`/var/lib/mlocate/`）中查找数据，查找速度也非常快。但由于linux中的数据库默认是每天更新一次，所以当要查找在数据库更新之后才新建立的文件时，它会告诉你此文件不存在。这是一个不好的地方，但还有补救措施，可以手动更新数据库

```shell
$ updatedb
```

updatedb 会根据 `/etc/updatedb.conf` 的设置取查找系统硬盘内的文件，并更新 `/var/lib/mlocate`内的数据库文件。由于它是将数据写入数据库文件，所以需要root权限。并且因为 updatedb 是查找硬盘文件，所以可能执行速度较慢。

还有一点，locate 是根据用户输入的关键字与数据库的记录进行匹配，所以可以不像 whereis 一样必须输入完整的文件名，只需输入关键字即可。

```shell
# 用法
$ locate [-ir] <keyword>

# 选项与参数
-i	: 忽略大小写差异
-r	: 后面可接正则表达式的匹配方式
-c	: 只显示文件数量
-l	: 仅输出几行
-S	: 输出locate所使用的数据库文件的相关信息，包括文件目录数量

# 查找关键字api，并仅显示10行
$ locate -l 10 api

# 显示查询的数据库文件信息
$ locate -S
```



## find

find 的功能十分强大，不但能根据文件名查找文件，还能根据时间、用户、权限、文件大小等条件查找文件，并且还可以指定查找的目录（连同子目录）进行查找。但由于它是针对硬盘进行查找，十分消耗硬盘资源，速度很慢，不推荐使用，显然whereis和locate是优先选择。



### 选项与参数

|     选项与参数     |                             含义                             |
| :----------------: | :----------------------------------------------------------: |
|   -name filename   |                      根据文件名查找文件                      |
| -size [+-]SIZE[ck] |     查找大于(+)或小于(-)SIZE大小的文件，c为bytes，k为kb      |
|    -mtime [+-]n    |      在n天之前(+)，第n天中，n天之内(-)修改过内容的文件       |
|     -type TYPE     |       根据文件类型查找，类型有**f, d**, b, c, l, s, p        |
|   -perm [-/]mode   | 查找刚好等于mode，全部囊括mode（-）和包含任一mode权限（/）的文件，mode为数字形势 |
|    -atime [+-]n    |                     同上，被读取过的文件                     |
|    -ctime [+-]n    |                    同上，修改过状态的文件                    |
|    -newer file     | 列出比 file 还要新的文件，file 必须包含路径（绝对相对均可）  |
|     -user name     |                  根据用户名查找该用户的文件                  |
|       -uid n       |             根据用户ID（为数字）查找该用户的文件             |
|    -groud name     |                             同上                             |
|       -gid n       |                             同上                             |
|      -nouser       |             查找文件的拥有者不在 /etc/passed 中              |
|      -nogroup      |                             同上                             |
|     -exec cmd      |                可接其他命令来处理查找到的结果                |



### 用法示例

```shell
$ find [PATH] [option] [action]

# 在系统下查找api.md文件
$ find / -name api.md

# 在当前目录下查找大于20b的文件
$ find . -size +20c

# 在当前目录下查找类型为目录的文件
$ find . -type d

# 在家目录下查找三天内修改过内容的文件
$ find /home -mtime -3

# 查找属于lawler的文件
$ find . -user lawler

# 查找权限等于0664的文件
$ find . -perm 0664

# 在上一层目录下查找比x.md新的文件且列出详细信息，并将结果输出到a.md文件中
$ find ../ -newer ./x.md -exec ls -l {} > a.md \;
```

`-exec` 参数中，`{}` 代表的是 find 查找到的结果，最后的 `\;` 代表命令的结束

