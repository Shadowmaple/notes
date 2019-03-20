# 装饰器

## 为什么要用装饰器？

先看一个简单的例子
```
def foo():
    print("I'm foo.")
    
foo()
```
之后有了一个新需求，想要在其中添加时间，你可以这样直接添加：
```
import time

def foo():
    t = time.strftime('%H:%M:%S', time.localtime())
    print('It\'s {} now.'.format(t))
    print("I am foo.")

foo()
```
先来看下效果
```
It's 15:15:15 now.
I am foo.
```
成功达成目的，但是这样的话，我们每次都要修改函数代码，如果有多个函数的话，效率极为低下。那么这种方式呢？
```
import time
def get_time(func):
    t = time.strftime('%H:%M:%S', time.localtime())
    print('It\'s {} now.'.format(t))
    func()
    
def foo():
    print("I am foo.")
    
get_time(foo)
```
当然可以。但是这种代码已经破坏了原有的代码逻辑，执行方式改变了，之前是执行运行foo()，而现在却只能运行get_time(foo)。那么有没有更好的方式呢？答案是装饰器。

## 什么是装饰器？
简单来讲，装饰器就是一个在不破坏原有代码逻辑、结构的情况下对既有函数方法进行附加功能的函数或类。它的本质就是函数或类。

### 简单的装饰器
首先要认识到在python中函数是一等公民，即函数也是对象，也是可以作为参数进行传递

```
def get_time(func):
    def wrapper():
        t = time.strftime('%H:%M:%S', time.localtime())
        print('It\'s {} now.'.format(t))
        return foo()
    return wrapper

def foo():
    print("I am foo.")
    
foo = get_time(foo)
foo()
```
函数get_time就是装饰器，它把执行真正业务方法的func包裹在函数里面，看起来像foo被get_time装饰了。
### 语法糖
@符号是装饰器的语法糖，在定义函数的时候使用，避免再一次赋值操作
```
def get_time(func):
    def wrapper():
        t = time.strftime('%H:%M:%S', time.localtime())
        print('It\'s {} now.'.format(t))
        return foo()
    return wrapper

@get_time               #加上语法糖，=> foo = get_time(foo)
def foo():
    print("I am foo.")
```
### 装饰器的调用顺序
```
@a
@b
@c
def foo():
    pass
```
装饰器的调用顺序是从下到上的，即`foo = a(b(c(foo)))`

## 带参数的装饰器
装饰器的语法允许我们在调用时，提供其它参数。这样，就为装饰器的编写和使用提供了更大的灵活性。
### 关于*args, **kwargs 
我们所要装饰的函数有很多是需要参数的，所以说我们可能会想到在wrapper()直接添加参数，但是如果函数所需要的参数个数是不确定的，那如果每次都修改所定义的装饰器，那么装饰器便失去了其普适性。所以引入了`*args` 和`**kwargs`支持接受动态参数。

+ `*args`表示任何多个无名参数，它是一个tuple
+ `**kwargs`表示关键字参数，它是一个dict.

```
def get_time(name):
    def decorator(func):
        def wrapper(*args, **kwargs):
            t = time.strftime('%H:%M:%S', time.localtime())
            print('It\'s {} now.'.format(t))
            print('I am {}.'.format(name))
            return func(*args, **kwargs)
        return wrapper
    return decorator

@get_time(name='Shadow')
def foo(name='Nick'):
    print("Hello, I'm %s." %name)

foo('Mark')
```

## 类装饰器
相比函数装饰器，类装饰器具有灵活度大、高内聚、封装性等优点。使用类装饰器主要依靠类的`__call__`方法，当使用@形式将装饰器附加到函数上时，就会调用此方法。
```
class Foo(object):
    def __init__(self, func):
        self._func = func

    def __call__(self):
        print ('class decorator runing')
        self._func()
        print ('class decorator ending')

@Foo
def bar():
    print ('bar')

bar()
```


## 装饰器的弊端
虽然装饰器实现了我们的功能，但是借助这个[网站](http://www.pythontutor.com/visualize.html#mode=edit)来观察返回的函数foo，我们会发现其名称将不会再是原函数的名称,即它的原信息发生了改变，这说明装饰器对原函数造成了影响，这是我们不愿意见到的。所以我们使用python的`functools`包中一个装饰器`wraps`来消除这个副作用。

```
from functools import wraps
import time

def get_time(name):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            t = time.strftime('%H:%M:%S', time.localtime())
            print('It\'s {} now.'.format(t))
            print('I am {}.'.format(name))
            return func(*args, **kwargs)
        return wrapper
    return decorator

@get_time(name='Shadow')
def foo(name='Nick'):
    print("Hello, I'm %s." %name)

foo('Mark')
```

## 装饰器的执行顺序
装饰器的一个关键特性是，它们在被装饰的函数定义之后立即运行。
```
def decorator(func):
    print("running decorator")
    return func
    
@decorator
def foo():
    print("running foo")

foo()
```
这就有运行和导入之分。导入时：
```
>>> import decorator
running decorator
```
foo()只有在运行时才会运行

## 闭包
明确三个变量：全局变量，局部变量和自由变量。

自由变量，指未在本地作用域中绑定的变量

```
def decorator(func):
    lt = ['abc']
    def wrapper():
        print(lt)
        return func
    return wrapper
```
在上述代码中，在`wrapper`中`lt`即是自由变量。从`lt = ['abc']`到`return func`之间（包括）即是闭包。

闭包是一种函数，它会保留定义函数时存在的自由变量的绑定，这样调用函数时，虽然定义作用域不可用了，但是仍能使用那些绑定。但是只有嵌套在其他函数中的函数才可能需要处理不在全局作用域中的外部变量。

