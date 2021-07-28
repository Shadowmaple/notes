## channel

### closed channel

被关闭的 channel 不能再向其中发送内容，否则会 panic

```go
ch := make(chan int)
close(ch)
ch <- 1 // panic: send on closed channel
```

在不确定是否还有 goroutine 需要向 channel 发送数据时，不要贸然关闭 channel。

但仍可以从已经 closed 的 channel 中接收值：

```go
ch := make(chan int)
close(ch)
x := <-ch
```

如果 channel 中有值，这里特指带 buffer 的 channel，那么就从 channel 中取，如果没有值，那么会返回 channel 元素的 0 值。

区分是返回的零值还是 buffer 中的值可使用 comma, ok 语法：

```go
x, ok := <-ch
```

若 ok 为 false，表明 channel 已被关闭，所得的是无效的值。

### nil channel

不进行初始化，即不调用 make 来赋值的 channel 称为 nil channel：

```go
var a chan int
```

关闭一个 nil channel 会直接 panic

```go
var a chan int
close(a) // panic: close of nil channel
```

### close channel principles

一条广泛流传的关闭 channel 的原则：

>   don't close a channel from the receiver side and don't close a channel if the channel has multiple concurrent senders.
>
>   不要从接收端关闭channel，也不要关闭一个有多个并发发送方的channel。

以及更本质的原则：

>   don't close (or send values to) closed channels.
>
>   不要关闭或发送数据到一个已关闭的channel，因为这会导致pannic。

根据 sender 和 receiver 的个数，分下面几种情况：

1.  一个 sender，一个 receiver
2.  一个 sender， M 个 receiver
3.  N 个 sender，一个 reciver
4.  N 个 sender， M 个 receiver

#### 一个sender和一或多个reveiver

只有一个 sender 的情况，直接从 sender 端关闭就好了。

#### N个sender, 一个reciver

第 3 种情形下，优雅关闭 channel 的方法是：

>   the only receiver says "please stop sending more" by closing an additional signal channel。

解决方案就是增加一个传递关闭信号的 channel，receiver 通过信号 channel 下达关闭数据 channel 指令。senders 监听到关闭信号后，停止接收数据。

```go
func main() {
    rand.Seed(time.Now().UnixNano())

    const Max = 100000
    const NumSenders = 1000

    dataCh := make(chan int, 100)
    stopCh := make(chan struct{})

    // senders
    for i := 0; i < NumSenders; i++ {
        go func() {
            for {
                select {
                case <- stopCh:
                    return
                case dataCh <- rand.Intn(Max):
                }
            }
        }()
    }

    // the receiver
    go func() {
        for value := range dataCh {
            if value == Max-1 {
                fmt.Println("send stop signal to senders.")
                close(stopCh)
                return
            }

            fmt.Println(value)
        }
    }()

    select {
    case <- time.After(time.Hour):
    }
}
```

上面的代码并没有明确关闭 dataCh。在 Go 语言中，对于一个 channel，如果最终没有任何 goroutine 引用它，不管 channel 有没有被关闭，最终都会被 gc 回收。所以，在这种情形下，所谓的优雅地关闭 channel 就是不关闭 channel，让 gc 代劳。

#### N 个 sender, M 个 receiver

对于N 个 sender 和 M 个 receiver，优雅关闭 channel 的方法是：

>   any one of them says "let's end the game" by notifying a moderator to close an additional signal channel

需要添加一个关闭 channel 的中间方，即 N 个 senders 需要将关闭请求发送给中间方，由中间方来关闭。

最后，如果确定不会有 goroutine 在通信过程中被阻塞，也可以不关闭 channel，等待 GC 对其进行回收。

### 源码分析

#### hchan

hchan 是 channel 在 runtime 中的数据结构

```go
// channel 在 runtime 中的结构体
type hchan struct {
    // 队列中目前的元素计数
    qcount uint // total data in the queue
    // 环形队列的总大小，ch := make(chan int, 10) => 就是这里这个 10
    dataqsiz uint // size of the circular queue
    // void * 的内存 buffer 区域
    buf unsafe.Pointer // points to an array of dataqsiz elements
    // sizeof chan 中的数据
    elemsize uint16
    // 是否已被关闭
    closed uint32
    // runtime._type，代表 channel 中的元素类型的 runtime 结构体
    elemtype *_type // element type
    // 发送索引
    sendx uint // send index
    // 接收索引
    recvx uint // receive index
    // 接收 goroutine 对应的 sudog 队列
    recvq waitq // list of recv waiters
    // 发送 goroutine 对应的 sudog 队列
    sendq waitq // list of send waiters

    // lock protects all fields in hchan, as well as several
    // fields in sudogs blocked on this channel.
    //
    // Do not change another G's status while holding this lock
    // (in particular, do not ready a G), as this can deadlock
    // with stack shrinking.
    lock mutex
}
```





>   https://github.com/cch123/golang-notes/blob/master/channel.md

### 参考

-   [码农桃花源-如何优雅地关闭 channel](https://qcrao91.gitbook.io/go/channel/ru-he-you-ya-di-guan-bi-channel)
-   [Golang-notes/channel](https://github.com/cch123/golang-notes/blob/master/channel.md)

