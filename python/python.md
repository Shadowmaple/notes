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

```python
>>> import getpass
>>> a = getpass.getpass('密码：')
密码：
>>> a
'1220'
```

## encode:
作用：以指定的编码格式编码字符串
效果：

```python
>>> a = '五月'
>>> a.encode('GB2312')
b'\xce\xe5\xd4\xc2'
```


## dir()
显示模块中的函数及方法


## datetime模块
+ datetime.now	本地时间
+ datetime.utcnow	世界时间

## assert 断言
格式 ： `assert 判断语句, “报错语句”`
```python
>>> assert 1>4, "error"     # 错误报错，输出错误语句
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError: error
>>> assert 1<3, "error"     # 正确则不输出
>>>
```
## hasattr()
用于判断对象是否包含对应的属性。

用法：`hasattr(object, name)`

如果对象有该属性，则返回True，反之相反

# python中if-else的简洁写法

第一种：普通写法
```python
a, b, c = 1, 2, 3
if a>b:

    c = a

else:

    c = b
```

第二种：一行表达式，为真时放 if 前

```python
c = a if a>b else b
```

第三种：二维列表，利用大小判断的0，1当作索引

```python
c= [b, a][a > b]
```

 第四种：传说中的黑客，利用逻辑运算符进行操作，都是最简单的东西，却发挥无限能量啊
```python
c = (a>b and [a] or [b])[0]
# 改编版
c = (a>b and a or b)
```
第四种最有意思了，

利用and 的特点，若and前位置为假则直接判断为假。

利用 or的特点，若or前位置为真则判断为真。


