# 木犀后端星计划

---

欢迎来到木犀星计划后端组的第二阶段，经过第一阶段的学习，相信同学们已经安装好了linux虚拟机，那么这次我们将尝试在linux终端上进行操作，以及学习使用文本编辑器——vim。

# Linux终端命令
linux的command何其繁多，这里只介绍一些比较常用的文件目录类命令。事先说明，在windows中的文件夹在linux中被称为目录，两者的性质其实差不多，只是名称不一样

首先，进入终端（一个小黑框）：Ctrl+Alt+t 

## ls : 显示文件列表
+ `ls`命令用于显示特定目录下的文件列表，可以直接显示当前目录下的文件列表，也可以加上一个目录名显示该目录下的文件列表
+ 还有其它可选参数比如： `-a `(显示全部文件，包括隐藏文件), `-l` (显示文件的属性)
```shell
# 展示当前目录下的文件列表
lawler@maple:~$ ls

# 比如当前目录下有muxi这个目录，显示muxi目录下的文件
lawler@maple:~$ ls muxi

# 显示全部文件，包括隐藏文件
lawler@maple:~$ ls -a

# 显示文件属性
lawler@maple:~$ ls -l
```

## mkdir : 新建目录
`mkdir` 用于在当前工作目录下新建一个目录
```shell
# 新建名为muxi的目录，当然如果已经存在那么就会报错
lawler@maple:~$ mkdir muxi
```

## cd : 切换当前目录
`cd`命令用于改变当前的工作目录
```shell
# 当前是"~"目录，即家(home)目录，进入muxi文件夹
lawler@maple:~$ cd muxi
lawler@maple:~/muxi$ 

# 退回上一层目录
lawler@maple:~/muxi$ cd ..
lawler@maple:~$ 
```

## touch : 新建一个文件
`touch` 用于在当前工作目录下新建一个文件
```shell
# 新建一个muxi.txt文件
lawler@maple:~$ touch muxi.txt
```
要想编辑该文件的内容，就要用到文本编辑器，其中一个便是Vim，这部分我会留到待会细说

## rm : 删除文件或目录
+ `rm` 是一个非常强大的command，它用于删除文件和目录，但要注意，在linux终端下执行的删除操作是不可逆的，也就是不可撤销的，所以用起`rm`务必慎重！
+ 常用的可选参数：`-i` (删除前提示确认)，`-rf` (递归删除目录)
```shell
# 删除muxi.txt文件
lawler@maple:~$ rm muxi.txt

# 删除前有一个确认的提示
lawler@maple:~$ rm -i muxi.txt
rm: remove regular empty file 'muxi.txt'?
# 这时如果确认便输入y (yes)，取消则输入n (no)

# 删除目录及其下的子目录、文件
lawler@maple:~$ rm -rf muxi
```

## mv : 移动文件位置或重命名
`mv` 有移动文件/目录位置和更名两个功能
```shell
# 将muxi.txt重命名为ccnu.txt
lawler@maple:~$ mv muxi.txt ccnu.txt

# 将ccnu.txt从家目录移入muxi目录中
lawler@maple:~$ mv ccnu.txt muxi
```

## cp : 复制文件
`cp` 可以将当前目录下的一个或多个文件复制到目标目录中
```shell
# 新建一个muxi.md文件并将其复制到muxi目录中
lawler@maple:~$ touch muxi.md
lawler@maple:~$ cp muxi.md muxi
# 可以试试ls命令，发现无论在~目录还是muxi目录都存在muxi.md文件
```

## shutdown : 关机
+ `shutdown` 作为关机命令，默认为一分钟后关机
+ 可选参数：`-r` (重启)，`-c` (取消关机)
```shell
# 关机，默认一分钟后生效
lawler@maple:~$ shutdown
# 取消关机
lawler@maple:~$ shutdown -c

# 也可以选择立即关机
lawler@maple:~$ shutdown now

# 重启
lawler@maple:~$ shutdown -r
```

## 其它
具体的命令用法可以通过`--help` 或 `-h`查看
```shell
lawler@maple:~$ cd --help
```

## 实际操练
单纯只是看的话，学习的效率不会很高，所以还是通过一些练习来掌握吧。学长在此布置了四道十分简单的题目，可以尝试着练习一下
1. 新建一个文件 muxi.md 和一个目录 muxiBox
2. 将 muxi.md 复制到 muxiBox 中
3. 进入muxiBox，将 muxi.md 更名为 ccnu.txt
4. 删除muxi.md文件

# 强大的文本编辑器——Vim
## 简介
命令行模式下的文本编辑器很多，有emacs、pico、nano、joe与vim等，但为什么一定要学vim呢？有以下几点原因：
> + 所有的UNIX-like系统都会内置vim文本编辑器，其他的文本编辑器不一定会存在
+ 很多软件的编辑接口都会主动调用vim（如crontab、visudo等）
+ vim具有程序编辑的能力，可以主动地以字体颜色辨别语法的正确性，方便程序设计
+ 因为程序简单，编辑速度相当快速

现在就来介绍一下vim的一些基本操作

## 基础操作
不出意外的话，你的系统已自带vim，就不用去安装了

用vim打开一个文件的命令如下
```
$ vim [filename]
```
当这个文件不存在时，它会自动帮你创建

例如，打开muxi.txt文件
```
$ vim muxi.txt
```
vim 大体上分为三个模式：一般命令模式、编辑模式和命令行模式。一开始进入vim的时候这时是一般命令模式，这时并不能编辑，而要想进入编辑模式，主要方式就是按下`i`，当你发现左下角出现`insert`或是`插入`时，就可以确定现在正处于编辑模式中了，这时你可以按下键进行常规的打字输入操作了。

而要想退出vim编辑器，则要先按下`Esc`键退回命令模式，然后输入`:q`即可退出，而要保存则要输入`:w`，具体看下表吧。

| command | 效果 |
| :---: | :---: |
| :q | 退出 |
| :q! | 强制退出，不保存修改 |
| :w | 保存修改 |
| :wq | 保存并退出 |
| ZZ | 保存并退出 |

## 命令模式的操作
vim与传统编辑器最大的不同就是纯靠键盘操控，比如光标定位操作，批量复制删除等。而这些操作大多数都是在一般命令模式下完成的。

一般命令模式（命令模式，普通模式），就是一开始打开的模式，在这个模式下，你可以通过键盘十分方便快速地进行定位、（批量）复制删除等操作。

### 光标移动与定位
在一般命令模式下，你只需通过上下左右的方向键移动即可，当然也可以用如下方式进行移动：
```
h       左
j       下
k       上
l       右
```
想快速定位可参考如下：
```
gg      回到顶部
G       最后一行
[n]G    回到第n行
```

### 复制、粘贴、删除
就先说下最为简单的复制粘贴和删除操作好了
```
yy      复制此行
dd      剪切此行
p       粘贴先前复制或剪切的内容
y[n]    复制n行
d[n]    粘贴n行
```
万一操作错误，想撤回怎么办？按下`u`就好了。

## 实际操练
这些命令大体上可以满足基本编辑需要了，其它的操作就自己去需要自己去学习了，下面布置了几个任务，可以尝试着去完成（注意在终端上使用vim操作）：
1. 新建一个名为 muxi.md 的文件，并用vim打开
2. 将本文内容复制入编辑器中（ps: 终端的粘贴快捷键是Ctrl+Shift+v）
3. 将实际操练这段内容剪切到最后一行
5. 撤销先前操作
6. 在命令行模式下查找“vim”这条字符串，并全部替换为"vi"
7. 保存并退出

第6道题是需要你自己去搜索学习的，加油！

# 最后的话
linux的终端命令和vim的操作指令当然不只这一些，学长也不会介绍很多，我们只是充当一个引导者，更多的还是看你们自己。下面提供了一些文档教程（你可以去搜更好的教程），期望你们能走得更远。

[Ubuntu 终端命令的使用](https://blog.csdn.net/hello_new_life/article/details/75099249)
[简明 VIM 练级攻略](https://coolshell.cn/articles/5426.html)
[Vim入门基础](https://www.jianshu.com/p/bcbe916f97e1)
