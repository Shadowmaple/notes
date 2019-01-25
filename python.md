# 问题

## 检查python模块是否安装
>$ python3 -c "import requests"


## python3安装pip的方法之一
1. 点击[链接](https://bootstrap.pypa.io/get-pip.py)，并下载`get-pip.py`文件;
2. 文件下载完成之后，cd到当前目录，并进行安装，`python3 get-pip.py`命令（如有必要使用超级权限）


# 模块与方法
## getpass模块
作用：密码输入不显示
效果：

    >>> import getpass
    >>> a = getpass.getpass('密码：')
    密码：
    >>> a
    '1220'

## encode:
作用：以指定的编码格式编码字符串
效果：

    >>> a = '五月'
    >>> a.encode('GB2312')
    b'\xce\xe5\xd4\xc2'


## dir()
显示模块中的函数及方法


## datetime模块
+ datetime.now	本地时间
+ datetime.utcnow	世界时间
