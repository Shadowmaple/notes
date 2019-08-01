# Linux的文件压缩

# 文件压缩

在现在的linux中，常用的单个文件压缩软件有gzip，bzip2和xz，就压缩比而言，gzip < bzip2 < xz，但相对的，压缩时间 gzip < bzip2 < xz。 

## gzip

gzip 是一种使用较为广泛的压缩命令，它可以解开 compress、zip和gzip等软件所压缩的文件，其压缩后的文件名为 *.gz

```shell
$ gzip [option] {filename}

选项和参数：
-c	: 将压缩数据输出到屏幕上
-d	: 解压缩
-k	: 保留原始文件
-v	: 显示压缩信息（压缩比等）
-n	: 1-9的数字，代表压缩等级，1最差，9最好，默认为6
-l	: 列出压缩文件的相关信息

# 压缩文件
# 这样之后压缩的文件会覆盖原始文件
$ gzip example.md

# 压缩文件，显示压缩信息，并保留源文件
# 第一种方法
$ gzip -kv example.md
# 第二种方法，使用数据流重定向
$ gzip -cv example.md > example.md.gz

# 解压缩文件
$ gzip -dv example.md.gz

# 使用9级压缩
$ gzip -9v example.md

# 显示压缩文件的信息
$ gzip -l example.gz
```

一般来说无论压缩还是解压缩，默认都是替换原有文件，要想保留原始文件就必须另加选项或参数。

至于压缩等级，虽然9级压缩率最好，但是时间成本会比较高，酌情使用。



使用 zcat, zmore, zless 可以读取被压缩后的压缩文件的纯文本内容。使用 zgrep 可以在压缩文件中查找纯文本内的关键字。

```shell
$ zgrep "vim" example.md.gz
```



## bzip2

bzip2 是比 gzip 更好的压缩命令，其提供的压缩比更好，但相对的，它所花的压缩时间也会更久。bzip2 的用法与gzip基本相同，压缩后的文件名为 *.bz2

```shell
$ bzip2 [option] [filename]

选项和参数：
-c	: 将压缩数据输出到屏幕上
-d	: 解压缩
-k	: 保留原始文件
-v	: 显示压缩信息
-n	: 1-9的数字，代表压缩等级，1最差，9最好，默认为6
-l	: 列出压缩文件的相关信息

# 压缩文件，并保留原始文件
$ bzip2 -kv example.md

# 解压缩
$ bzip2 -dv example.md.bz2
```

同样的，bzip2 也有 bzat，bzmore，bzless 和 bzgrep 使用。

## xz

xz 是比 bzip2 更棒的一个压缩命令，同样的，压缩的时间成本也更高。其压缩文件为 *.xz，其用法与前两者基本相同，所以就不显示示例了。

```shell
$ xz [option] [filename]

选项和参数：
-c	: 将压缩数据输出到屏幕上
-d	: 解压缩
-k	: 保留原始文件
-v	: 显示压缩信息
-n	: 1-9的数字，代表压缩等级，1最差，9最好，默认为6
-l	: 列出压缩文件的相关信息
```

同样的，xz 也有 xzcat，xzmore，xzless和xzgrep。



# 打包压缩

gzip，bzip2和xz只是针对单个文件进行压缩，一般情况下不能针对一个目录。但是通过 tar 对一个目录进行打包就可以进行压缩。可以用 ta r打包再 用 gzip 等进行压缩，也可以将这两个步骤合并使用。

```shell
# 打包与压缩
$ tar [-z|-j|-J] [-cv] [-f 目标文件名] 被压缩文件名
# 查看打包文件
$ tar [-z|-j|-J] [-tv] [-f 文件名]
# 解压缩
$ tar [-z|-j|-J] [-xv] [-f 文件名] [-C 目录]

选项与参数：
-c	: 建立打包文件
-t	: 查看打包文件含有的文件
-x	: 解包或解压缩
----------------------------------------
-z	: 使用 gzip 进行压缩，文件名为 *.tar.gz
-j	: 使用 bzip2 进行压缩，文件名为 *.tar.bz2
-J	: 使用 xz 进行压缩，文件名为 *.tar.xz
----------------------------------------
-v	: 显示处理的文件信息
-f 	: 后要跟被处理的文件名
-C 	: 后跟目录名，在解压缩时解压至该目录
----------------------------------------
-p	: 保留数据原有权限和属性
-P	: 保留绝对路径
--exclude=FILE	: 在压缩过程中，忽略该文件


# 示例
# 打包
# 若在开头加上 time 会显示程序运行时间
$ tar -cv -f example.tar example

# 打包并用gzip压缩
$ tar -zcvf example.tar.gz example

# 查看打包文件中所含文件
# 若加上-v 则会显示文件完整（属性）信息
$ tar -ztf example.tar.gz

# 解压缩至当前目录
$ tar -zxvf example.tar.gz -C .
```

