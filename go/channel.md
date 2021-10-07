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

区分是返回的零值还是 buffer 中的值可使用 `comma, ok` 写法：

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

nil channel 会对**读写**进行永久**阻塞**（一个g会造成死锁），而不是pannic：

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

#### makechan函数

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

形如`c <- x` 的发送语法会被编译器转换成`chansend1`函数，而`chansend1`又是直接调用`chansend`函数：

```go
func chansend1(c *hchan, elem unsafe.Pointer) {
    chansend(c, elem, true, getcallerpc())
}
```

而 `select` 中的发送语法会被编译成`selectnbsend`函数，同样也是调用`chansend`函数，但不同的是传入的`block`字段是true，即非阻塞的，因为select要立刻判断，如果不能发送就下一个。

```go
//	select {
//	case c <- v:
//		... foo
//	default:
//		... bar
//	}
//
// as
//
//	if selectnbsend(c, v) {
//		... foo
//	} else {
//		... bar
//	}
//
func selectnbsend(c *hchan, elem unsafe.Pointer) (selected bool) {
	return chansend(c, elem, false, getcallerpc())
}
```



#### chansend函数

chansend 大致可以根据不同情况分为以下三部分：

1.  **直接发送**：若存在等待的接受者，直接调用`runtime.send`发送数据给阻塞的g；
2.  **数据写入缓冲区**：若缓冲区存在空闲空间，则将数据拷贝到缓冲区中；
3.  **阻塞发送**：若不存在缓冲区或已满，则挂起g，等待被唤醒。



chansend大致流程：

1.  判断是channel否为nil，是则永久挂起阻塞
2.  加锁
3.  判断是否closed，若是则pannic
4.  能否直接发送：
    1.  若存在等待的接受者，则调用`runtime.send`发送给它
    2.  在send函数中解锁返回
5.  缓冲区是否有空闲空间：
    1.  计算存储的位置，并进行数据拷贝；
    2.  更新hchan队列元数据，sendx, recvx, qcount 等字段；
    3.  解锁返回
6.  进行阻塞发送流程：
    1.  调用 `runtime.getg` 获取发送数据使用的 Goroutine；
    2.  执行 `runtime.acquireSudog` 获取 `runtime.sudog` 结构并设置这一次阻塞发送的相关信息，例如发送的 Channel、是否在 select 中和待发送数据的内存地址等；
    3.  将刚刚创建并初始化的 `sudog` 加入发送等待队列，并设置到当前 g 的 waiting 字段上，表示 g 正在等待该 sudog 准备就绪；
    4.  调用 `runtime.goparkunlock` 将当前的 g 状态从Grunning变为Gwaiting，陷入沉睡等待唤醒；（挂起应该会解锁？）
    5.  被调度器唤醒后，判断channel是否closed了
    6.  执行一些收尾工作，将一些属性置零并且释放 runtime.sudog 结构体；
    7.  返回



```go
// block参数为true表示在发送时是阻塞的，即如果不能发送是否需要阻塞等待，
// 只有在select时才会被编译成block为true的情况。
// 返回值true表示发送成功，false为发送失败
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // 向一个未初始化的chan发送
    // 如：var c chan int
	if c == nil {
		if !block {
			return false
		}
        // nil channel 发送数据会永远阻塞下去
        // PS：注意，会发生 panic 那种情况是 channel 被 closed 了，不是 nil channel
        // 挂起当前 goroutine
		gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

    // 非阻塞模式下，迅速判断是否可以发送
	if !block && c.closed == 0 && ((c.dataqsiz == 0 && c.recvq.first == nil) ||
		(c.dataqsiz > 0 && c.qcount == c.dataqsiz)) {
		return false
	}

	var t0 int64
    // TO DO：不清楚这个是干什么的……
	if blockprofilerate > 0 {
		t0 = cputicks()
	}

    // 加锁，保证线程安全
	lock(&c.lock)

    // 如果已经关闭了，panic
	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("send on closed channel"))
	}
    
    // -----------
    // 1.直接发送
    // -----------

    // 从等待的队列中取出第一个goroutine，
    // 若g不为空，则直接向g发送数据，忽略buffer缓冲区，即不用先放到buffer中
    // 能够取出接受者，说明此时缓冲区为空
	if sg := c.recvq.dequeue(); sg != nil {
		send(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true
	}
    
    // ----------------
    // 2. 缓冲区数据发送
    // ----------------

    // 当缓冲区还有空间，则将数据入队
	if c.qcount < c.dataqsiz {
		qp := chanbuf(c, c.sendx) // 计算下一个数据可以存储的位置
		// ...
		typedmemmove(c.elemtype, qp, ep) // 将数据拷贝到缓冲区中
        // 更新发送索引和数量
		c.sendx++
		if c.sendx == c.dataqsiz {
			c.sendx = 0
		}
		c.qcount++
		unlock(&c.lock)
		return true
	}
    
    // --------------
    // 3. 阻塞发送阶段
    // --------------

    // 如果非阻塞的，直接返回false，表示发送失败
    // 阻塞模式的还要等待
	if !block {
		unlock(&c.lock)
		return false
	}

	// Block on the channel. Some receiver will complete our operation for us.
    // 在 channel 上阻塞，receiver 会帮我们完成后续的工作
	gp := getg() // 获取当前发送的goroutine指针
    // 获取sudog结构，并设置这一次阻塞发送的相关信息
    // 不同版本有差别，这里是1.14.3
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	// No stack splits between assigning elem and enqueuing mysg
	// on gp.waiting where copystack can find it.
	mysg.elem = ep
	mysg.waitlink = nil
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.waiting = mysg
	gp.param = nil
    // 将sudog放入channel的发送等待队列
	c.sendq.enqueue(mysg)
    // 挂起goroutine，状态 Grunning -> Gwaiting
    // To Do：解锁呢？
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)
	// Ensure the value being sent is kept alive until the
	// receiver copies it out. The sudog has a pointer to the
	// stack object, but sudogs aren't considered as roots of the
	// stack tracer.
    // To Do：？？？
	KeepAlive(ep)

    // 对比当前的sudog是否是goroutine上的sudog
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	if gp.param == nil {
		if c.closed == 0 {
			throw("chansend: spurious wakeup")
		}
        // 唤醒后发现channel被关了
		panic(plainError("send on closed channel"))
	}
	gp.param = nil
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	mysg.c = nil
	releaseSudog(mysg) // 释放sudog结构体
	return true
}
```

#### send函数

向一个空缓冲区的channel发送会调用`runtime.send`函数：

1.  形如`x = <-ch`，若要接收的变量x不为nil，即存在分配的地址，则调用 `sendDirect` 将数据直接拷贝到 x 所在的内存地址上；
2.  调用 `runtime.goready` 将等待接收数据的 g 标记成可运行状态，并把该 g 放到发送方所在的处理器的 runnext 上等待执行，该处理器在下一次调度时会立刻唤醒数据的接收方；

```go
// 向一个空缓冲区的channel执行发送操作
// eq是要发送的数据地址，必须是非空且指向堆空间或某个调用栈
// sg是接受者的sudog，等待唤醒，且必须是已经在c的等待队列中的
// channel c 必须是无数据的且上锁的
// skip??
func send(c *hchan, sg *sudog, ep unsafe.Pointer, unlockf func(), skip int) {
	// raceenabled的一些操作，先忽略
    // ……

    // 形如 x = <-ch，且 x 不为 nil，则直接拷贝数据到 x 的地址空间
    // 形如 <-ch，则不用拷贝数据
	if sg.elem != nil {
		sendDirect(c.elemtype, sg, ep)
        // TO DO：为什么要设为nil？为了防止多个channel重复拷贝？
		sg.elem = nil
	}
	gp := sg.g // 获取goroutine地址
	unlockf() // 解锁
	gp.param = unsafe.Pointer(sg) // ？？
	if sg.releasetime != 0 {
		sg.releasetime = cputicks()
	}
    // 将goroutine设为Grunnable状态……
	goready(gp, skip+1)
}
```



发送数据的过程中包含几个会触发 Goroutine 调度的时机：

1.  发送数据时发现 Channel 上存在等待接收数据的 g，立刻设置处理器的 runnext 属性，但是并不会立刻触发调度；
2.  发送数据时并没有找到接收方并且缓冲区已经满了，这时会将自己加入 Channel 的 sendq 队列并调用 `runtime.goparkunlock` 触发 g 的调度让出处理器的使用权；



TO DO：goparkunlock 挂起做了什么？



### receive

从channel接收数据有三个写法：

-   `<-c`：被编译成 `chanrecv1`函数
-   `x <- c`：被编译成 `chanrecv1`函数
-   `x, ok <- c`：同样被编译成 `chanrecv2`函数

```go
// entry points for <- c from compiled code
func chanrecv1(c *hchan, elem unsafe.Pointer) {
	chanrecv(c, elem, true)
}

func chanrecv2(c *hchan, elem unsafe.Pointer) (received bool) {
	_, received = chanrecv(c, elem, true)
	return
}
```

无论`chanrecv1`还是`chanrecv2`，都是调用 `chanrecv`方法

而在`select`下使用的会被编译成`selectnbrecv`函数，同样内部调用`chanrecv`方法，也是block为true的区别：

```go
//	select {
//	case v = <-c:
//		... foo
//	default:
//		... bar
//	}
//
// as
//
//	if selectnbrecv(&v, c) {
//		... foo
//	} else {
//		... bar
//	}
//
func selectnbrecv(elem unsafe.Pointer, c *hchan) (selected bool) {
	selected, _ = chanrecv(c, elem, false)
	return
}
```



#### chanrecv函数

chanrecv方法也可以根据情况大致分为四个部分：

1.  channel关闭
2.  直接接收：存在等待的发送者；
3.  缓冲区有数据接收：缓冲区不为空，从缓冲区队头拷贝数据；
4.  阻塞接收：缓冲区为空且无发送者，挂起阻塞等待唤醒。



chanrecv流程：

1.  判断chan是否为nil，为nil则挂起阻塞；
2.  加锁
3.  是否被关闭了且缓冲区无数据：
    1.  解锁，将接受变量置零值
    2.  返回true, false
4.  是否存在接收者
    1.  调用`runtime.recv`方法直接从接受者获取数据
    2.  返回true, true
5.  缓冲区是否存在数据
    1.  从队列头部获取数据
    2.  若接收变量不为nil，则拷贝数据到该地址空间上，否则忽略
    3.  返回true, true
6.  阻塞接收流程：
    1.  获取goroutine和sudog，设置一些参数
    2.  将goroutine放入channel的等待接收队列
    3.  挂起goroutine
    4.  被唤醒后进行一些收尾工作，重置goroutine的一些字段，释放sudog结构
    5.  判断返回值closed
    6.  返回true, !closed



```go
// eq：等待接收的数据地址，
//   - 对于形如<-c情况，eq为nil，直接忽略接收的数据
//   - eq不为nil，则必须指向堆或调用栈（也就是说逃逸分析会把接收的变量放到堆上？）
// block：和send一样，表示是否阻塞的意思，只有在select时会用到false
// 返回值selected表示是否被选择上了/接收到了数据，chan关闭也为true，用于select选择语法
// 返回值received表示是否成功接收到了有效数据，若chan关闭则返回false
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
    // ...

	if c == nil {
		if !block {
			return
		}
		gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

    // 非阻塞模式下迅速判断能否接收到数据，无需加锁
    // 一堆注释……
	if !block && (c.dataqsiz == 0 && c.sendq.first == nil ||
		c.dataqsiz > 0 && atomic.Loaduint(&c.qcount) == 0) &&
		atomic.Load(&c.closed) == 0 {
		return
	}

    // 干什么用的？？……
	var t0 int64
	if blockprofilerate > 0 {
		t0 = cputicks()
	}

	lock(&c.lock)

    // 如果chan被关闭且无数据在缓冲区中，
    // 那么会解锁，并清除ep指针中的数据并返回（也就是将ep设为零值返回false）
	if c.closed != 0 && c.qcount == 0 {
        // ...
		unlock(&c.lock)
		if ep != nil {
			typedmemclr(c.elemtype, ep)
		}
		return true, false
	}
    
    // ------------
    // 直接发送
    // ------------

    // 如果存在等待的发送者，则直接从发送者中接收数据
	if sg := c.sendq.dequeue(); sg != nil {
		// Found a waiting sender. If buffer is size 0, receive value
		// directly from sender. Otherwise, receive from head of queue
		// and add sender's value to the tail of the queue (both map to
		// the same buffer slot because the queue is full).
		recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true, true
	}
    
    // -------------
    // 缓冲区接收数据
    // -------------

    // 缓冲区存在数据
	if c.qcount > 0 {
        // 直接从队列头部获取数据
		qp := chanbuf(c, c.recvx)
		// ...
        // 若ep不为nil，则将数据qp拷贝到ep的地址空间上，否则忽略
		if ep != nil {
			typedmemmove(c.elemtype, ep, qp)
		}
        // 清除qp的数据
		typedmemclr(c.elemtype, qp)
        // 更新hchan的一些元数据
		c.recvx++
		if c.recvx == c.dataqsiz {
			c.recvx = 0
		}
		c.qcount--
		unlock(&c.lock)
		return true, true
	}
    
    // ------------
    // 阻塞接收
    // ------------

	if !block {
		unlock(&c.lock)
		return false, false
	}

	// no sender available: block on this channel.
	gp := getg() // 获取goroutine
	mysg := acquireSudog() // 获取sudog结构体
    // to do：？？
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	// No stack splits between assigning elem and enqueuing mysg
	// on gp.waiting where copystack can find it.
	mysg.elem = ep
	mysg.waitlink = nil
	gp.waiting = mysg
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.param = nil
	c.recvq.enqueue(mysg)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanReceive, traceEvGoBlockRecv, 2)

	// someone woke us up
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
    // chan是否关闭，根据goroutine的param字段判断，
    // todo：前面设置为nil，这里是恒定nil吗？
	closed := gp.param == nil
	gp.param = nil
	mysg.c = nil
	releaseSudog(mysg)
	return true, !closed
}
```



#### recv函数

直接发送操作、对一个无缓冲区或满缓冲区的channel接收操作会调用`runtime.recv`方法：

1.  判断channel类型：

    -   如果是无缓冲区的 channel：
        -   调用 `runtime.recvDirect` 将 channel 发送队列中 Goroutine 存储的要发送的数据 elem 拷贝到要接收方的目标内存地址 ep 中；

    -   如果 channel 存在缓冲区：
        1.  将缓冲区队头的数据拷贝到接收方的内存地址；
        2.  将发送队列头goroutine的发送数据拷贝到缓冲区队尾中。

2.  运行时调用 `runtime.goready` 将当前处理器的 `runnext` 设置成发送数据的 Goroutine，在调度器下一次调度时将阻塞的发送方唤醒。



源码：

```go
// recv函数用于对一个满缓冲区的channel实行接收操作。
// 分为两部分：
// 1) 发送方将发送的数据被放入channel，同时这个发送方被唤醒继续执行；
// 2) 接收方要将接收的数据写入ep（接收的变量内存地址中）。
// 对于同步的channel（无缓冲），以上两部分的数据都是一样的；
// 对于异步的channel（有缓冲），接收方从缓冲区队头获取数据，同时发送方将发送的数据放入缓冲区队尾。
// channel 必须是满的且是上锁的，要唤醒的发送方必须已经进入channel的等待发送队列
// 非空的 ep 必须指向堆或某个调用栈
// skip？？
func recv(c *hchan, sg *sudog, ep unsafe.Pointer, unlockf func(), skip int) {
	if c.dataqsiz == 0 {
        // 如果是无缓冲区的channel，直接从发送方拷贝数据到接收方的内存地址中
		if ep != nil {
			// copy data from sender
			recvDirect(c.elemtype, sg, ep)
		}
	} else {
        // 如果是有缓冲区的channel，此时缓冲区队列已满，从队头获取要接收的数据，
        // 再将阻塞的发送方唤醒，将要发送的数据放入队尾，
        // 因为队列是满的，所以两者的位置相同。
		qp := chanbuf(c, c.recvx)
		// ...
        // 拷贝数据到发送方的内存地址中
		if ep != nil {
			typedmemmove(c.elemtype, ep, qp)
		}
        // 拷贝发送方的数据到相同的缓冲区位置中
		typedmemmove(c.elemtype, qp, sg.elem)
		c.recvx++
		if c.recvx == c.dataqsiz {
			c.recvx = 0
		}
		c.sendx = c.recvx // c.sendx = (c.sendx+1) % c.dataqsiz
	}
    // 无论哪种情况，发送方都已经成功发送数据了，所以设置下发送方，并将其唤醒
	sg.elem = nil
	gp := sg.g
	unlockf()
	gp.param = unsafe.Pointer(sg)
	if sg.releasetime != 0 {
		sg.releasetime = cputicks()
	}
	goready(gp, skip+1)
}
```



### close

`close(ch)`语法会被编译成`closechan`函数

#### closechan函数

`runtime.closechan`函数：

1.  判断为nil，panic：`close of nil channel`
2.  加锁
3.  判断是否已关闭，是则解锁并panic：`close of closed channel`
4.  设置`closed=1`已关闭
5.  循环，将所有等待的接收者放入需要释放的队列gList
6.  循环，将所有等待的发送者放入gList（这里被唤醒的发送者会panic）
7.  解锁
8.  将gList中所有阻塞的goroutine唤醒



源码：

```go
func closechan(c *hchan) {
	if c == nil {
		panic(plainError("close of nil channel"))
	}

	lock(&c.lock)
	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("close of closed channel"))
	}

    // ...

	c.closed = 1

    // 需要唤醒的goroutine序列
	var glist gList

    // 释放所有等待的接收者
	for {
		sg := c.recvq.dequeue()
		if sg == nil {
			break
		}
		if sg.elem != nil {
			typedmemclr(c.elemtype, sg.elem)
			sg.elem = nil
		}
		if sg.releasetime != 0 {
			sg.releasetime = cputicks()
		}
		gp := sg.g
		gp.param = nil
		// ...
		glist.push(gp)
	}

    // 释放所有等待的发送者，它们会panic
	for {
		sg := c.sendq.dequeue()
		if sg == nil {
			break
		}
		sg.elem = nil
		if sg.releasetime != 0 {
			sg.releasetime = cputicks()
		}
		gp := sg.g
		gp.param = nil
		// ...
		glist.push(gp)
	}
	unlock(&c.lock)

	// Ready all Gs now that we've dropped the channel lock.
    // 将所有等待的goroutine唤醒，从Gwaiting->Grunning
	for !glist.empty() {
		gp := glist.pop()
		gp.schedlink = 0
		goready(gp, 3)
	}
}
```





## 参考

-   [码农桃花源-如何优雅地关闭 channel](https://qcrao91.gitbook.io/go/channel/ru-he-you-ya-di-guan-bi-channel)
-   [Golang-notes/channel](https://github.com/cch123/golang-notes/blob/master/channel.md)
-   [channel底层原理](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/)

