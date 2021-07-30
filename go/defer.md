## defer

### 知识点

1.  defer后面必须跟函数，如 `defer fmt.Println("ok")`
2.  defer后面的函数在defer语句所在的函数结束时会被调用
3.  如果函数里面有多条defer指令，他们的执行顺序是反序，即后定义的defer先执行，栈的方式FILO
4.  defer与return执行顺序：return后的语句先，defer后。

```go
func returnFunc() int {
    fmt.Println("return func called")
    return 0
}

func foo() int {
    defer func() {
        fmt.Println("defer func called")
    }()
    return returnFunc()
}

/*
结果：
returnFunc called
defer func called
*/
```

6.  虽然先return再defer，但defer会影响返回值。比如以下程序，res的值为1，而不是2。

```go
func foo() (res int) {
    defer func() {
    	res = 1
    }()
    return 2
}
```

7.  defer 最大的功能是 panic 后依然有效，所以常用来捕获异常或关闭一些资源（比如锁、channel）

### 源码分析

https://github.com/cch123/golang-notes/blob/master/defer.md



### 参考

-   [Golang修养之路](https://www.kancloud.cn/aceld/golang/1958310#2_deferreturn_50)