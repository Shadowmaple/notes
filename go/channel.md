# channel

## create and init

```go
ch := make(chan int)
ch2 := new(chan int)
```

make和new都能创建，但是channel是引用类型的，应该用前者，后者返回的是`*chan int`，而且是未初始化的，即`nil channel`

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

nil channel 会对读写进行永久**阻塞**，而不是pannic：

```go
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send (nil chan)]:
main.main()
	/home/lawler/repository/notes/go/a.go:7 +0x3a
exit status 2
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



## 底层实现

### 基本结构

channel 的底层比较容易理解，主要就是一个 hchan 的结构体，它是一个循环队列，队头和队尾分别指向接受的数据和发送的数据。

另外，它有使用锁来保证其操作的原子性。

hchan 是 channel 在 runtime 中的数据结构：

```go
// channel 在 runtime 中的结构体
type hchan struct {
    qcount uint        // 队列中目前的元素总数
    dataqsiz uint      // 循环队列的长度，make(chan int, 10) => 就是这里这个 10
    buf unsafe.Pointer // 指向buffer数据缓冲区的指针
    elemsize uint16    // chan中元素类型的大小，如sizeof(int)
    closed uint32      // 是否已被关闭
    elemtype *_type    // 表示 channel 中的元素类型的指针
    sendx uint         // 发送索引，指向队头
    recvx uint         // 接收索引，指向队尾
    recvq waitq        // 等待接收数据的 goroutine 队列，g处于阻塞
    sendq waitq        // 等待发送数据的 goroutine 队列，g处于阻塞

    // lock protects all fields in hchan, as well as several
    // fields in sudogs blocked on this channel.
    //
    // Do not change another G's status while holding this lock
    // (in particular, do not ready a G), as this can deadlock
    // with stack shrinking.
    // 
    // 使用 runtime.mutex 来保护 chan 的线程安全，在 hchan 的各个领域，以及一些阻塞g的相关领域
    // 不要在持有锁的时候更改其它g的状态，尤其不要将一个g设为准备状态，因为这会在栈收缩的时候导致死锁
    // （可能理解会有偏差……）
    lock mutex
}

// 阻塞的等待goroutine队列，链表形式
type waitq struct {
	first *sudog
	last  *sudog
}
```





一些常量的定义：

```go
const (
	maxAlign  = 8
	hchanSize = unsafe.Sizeof(hchan{}) + uintptr(-int(unsafe.Sizeof(hchan{}))&(maxAlign-1))
	debugChan = false
)
```



### init

创建并初始化一个channel，使用的是`makechan`和`makechan64`方法，后者是为了应对缓冲区大小大于2\^64的情况（毕竟存储元素数量的类型是uint，最大是2\^64个），但在代码中有点迷，没明白啥意思……

主要是`makechan`这个函数（不同版本会有差异）：

```go
func makechan(t *chantype, size int) *hchan {
	elem := t.elem

	// compiler checks this but be safe.
	if elem.size >= 1<<16 {
		throw("makechan: invalid channel element type")
	}
	if hchanSize%maxAlign != 0 || elem.align > maxAlign {
		throw("makechan: bad alignment")
	}

    // 需要分配的内存大小
	mem, overflow := math.MulUintptr(elem.size, uintptr(size))
	if overflow || mem > maxAlloc-hchanSize || size < 0 {
		panic(plainError("makechan: size out of range"))
	}

	// Hchan does not contain pointers interesting for GC when elements stored in buf do not contain pointers.
	// buf points into the same allocation, elemtype is persistent.
	// SudoG's are referenced from their owning thread so they can't be collected.
	// TODO(dvyukov,rlh): Rethink when collector can move allocated objects.
    // 
    // 初始化hchan的一些字段，注释说明了一些字段需要在runtime时才会有值
    // 当buffer中的元素不包含指针的时候，hchan就不会包含和GC相关的信息
    // buf 指向同一块分配区，元素类型是持久的
    // SudoG 从它们占有的线程引入，所以无法收集它们的信息，需要在runtime时收集
	var c *hchan
	switch {
	case mem == 0:
		// Queue or element size is zero.
        // 如果 channel 的缓冲区大小为 0: ch := make(chan int)
        // 或者 channel 的元素大小为 0: struct{}{}
        // 分配一段hcan结构大小的内存
		c = (*hchan)(mallocgc(hchanSize, nil, true))
		// Race detector uses this location for synchronization.
		c.buf = c.raceaddr()
	case elem.ptrdata == 0:
        // 若元素不包含指针，则一次性为hchan和缓冲区大小分配内存
        // 为什么要这样？
        // 这种情况下 gc 不会对 channel 中的元素进行 scan？
		c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
		c.buf = add(unsafe.Pointer(c), hchanSize)
	default:
        // 元素包含指针，即是指针类型的元素时
        // 和上面那个 case 的写法的区别：调用了两次分配空间的函数 new/mallocgc
		c = new(hchan)
		c.buf = mallocgc(mem, elem, true)
	}

	c.elemsize = uint16(elem.size)
	c.elemtype = elem
	c.dataqsiz = uint(size)

	if debugChan {
		print("makechan: chan=", c, "; elemsize=", elem.size, "; dataqsiz=", size, "\n")
	}
	return c
}
```



### send



### receive



### close



## 参考

-   [码农桃花源-如何优雅地关闭 channel](https://qcrao91.gitbook.io/go/channel/ru-he-you-ya-di-guan-bi-channel)
-   [Golang-notes/channel](https://github.com/cch123/golang-notes/blob/master/channel.md)

