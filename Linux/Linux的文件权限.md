# Linux的文件权限

# 权限身份

三种身份：
1. 用户 User
2. 群组 Group
3. 其他人 Others

用户身份和用户组记录的文件：

+ /etc/passwd ： 存储系统账户、一般用户和root账户的相关信息（默认情况下）
+ /etc/shadow ： 记录个人密码
+ /etc/group ： 记录所有组名

# 文件属性
```shell
➜  HomeWorks 21:15:00 git:(master) ls -l
total 132
-rw-rw-r-- 1 lawler lawler 26041 4月  17 21:19 apiDoc.yaml
-rw-rw-r-- 1 lawler lawler  9355 4月  17 21:20 api.md
drwxrwxr-x 5 lawler lawler  4096 5月   4 14:58 app
-rw-r--r-- 1 lawler lawler 32768 4月  23 09:48 app.db
-rw-rw-r-- 1 lawler lawler 16384 4月  25 22:42 celerybeat-schedule
-rw-rw-r-- 1 lawler lawler   201 4月  16 15:22 celeryout.file
-rw-rw-r-- 1 lawler lawler   194 4月  16 08:08 celery_run.py
-rw-rw-r-- 1 lawler lawler  1199 4月  23 09:49 config.py
-rw-rw-r-- 1 lawler lawler    97 5月   4 23:44 manage.py
-rw-rw-r-- 1 lawler lawler   244 4月  16 15:19 myout.file
drwxrwxr-x 2 lawler lawler  4096 4月  23 09:49 __pycache__
-rw-rw-r-- 1 lawler lawler    55 3月  17 00:14 README.md
-rw-rw-r-- 1 lawler lawler   192 4月  17 21:48 requirement.txt
-rw-rw-r-- 1 lawler lawler   116 3月  30 17:36 run.sh
-rw-rw-r-- 1 lawler lawler   873 3月  17 12:00 test.py
```
第一栏：文件的类型和权限
第一个字符表示文件的类型，permssion
d是目录，【-】是文件，【l】是链接文件，【b】为设备文件里的可供存储的周边设备，【c】为设备文件里面的串行端口设备，如键盘鼠标（一次性读取设备）
接下来的字符
rwx：可读可写可执行（read, write, execute）
分为三组，分别是文件拥有者的权限，该用户组的权限，其他人的权限

第二栏：链接节点数，incode
第三栏：拥有者账号
第四栏：所属用户组
第五栏：容量大小，默认为 Bytes
第六栏：最近的修改时间。格式为日期（月/日）和时间，但若时间过久远，便会仅显示日期和年份。使用`ls -l --full-time`显示完整的时间格式
第七栏：文件名

# 权限意义
| 组件 | ｒ | w | x |
| :---: | :---: | :---: | :---: |
| 文件 | 读取文件内容 | 修改文件内容 | 执行文件 |
| 目录 | 读到文件名，读取目录结构列表 | 修改文件，包括增删文件、更名、移动 | 进入目录 |
注：
1. 在 Linux 下，[x] 权限决定文件能否被执行，这和 Windows 有所不同
2. 文件有 [w] 权限只能对文件内容进行修改，不包含该文件的删除
3. 若目录无 [r] 权限，则无法 list ，在某些系统和shell上甚至无法使用 [tab] 进行自动补全
4. 从网络上下载一个文件时，其属性和权限会发生改变

# 修改文件属性和权限
chgrp： 修改文件所属用户组
chown： 修改文件拥有者以及用户组
chmod： 修改文件权限

## chgrp

```shell
# 格式
$ chgrp [option] grouup filename

# -R 为递归修改
$ chgrp -R lawler blog/
```
## chown

```shell
# 格式
$ chown [option] user filename
$ chown [option] user:group filename    # 也可用小数点[.]来间隔

# 例子
# chown lawler:lawler test.c
```
## chmod
数字类型权限表示：`rwx` --> 421
符号类型权限表示：

+ `+ - =` --> 加入，移除，设置
+ `u g o a` --> user，group，others，all

```shell
# 格式
$ chmod [option] mode filename

# 例子
# 数字类型修改
$ chmod 654 test.sh     # 更改后为 -rw-r-xr--

# 符号类型修改
$ chmod u=rwx,go=rx test.sh     # 设置user为读写执行，用户组和其他为读执行
$ chmod -R a+x blog/       # 都加上可执行权限（该目录递归添加）
```
