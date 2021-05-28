# Go 并发模式

## CSP

CSP(communicating sequential processes)，通信顺序进程

>   "Do not communicate by sharing memory; instead, share memory by communicating."
>
>   不要使用共享内存来通信，而应该通过通信来共享内存。

Go 语言推荐我们使用通信来进行进程间同步消息。这样做有三点好处，来源于 [draveness](https://draveness.me/whys-the-design-communication-shared-memory) 的博客文章：

1.  首先，使用发送消息来同步信息相比于直接使用共享内存和互斥锁是一种更高级的抽象，使用更高级的抽象能够为我们在程序设计上提供更好的封装，让程序的逻辑更加清晰；
2.  其次，消息发送在解耦方面与共享内存相比也有一定优势，我们可以将线程的职责分成生产者和消费者，并通过消息传递的方式将它们解耦，不需要再依赖共享内存；
3.  最后，Go 语言选择消息发送的方式，通过保证同一时间只有一个活跃的线程能够访问数据，能够从设计上天然地避免线程竞争和数据冲突的问题；



## 并发模式

1.  for-select

```go
// 给异步执行加上超时机制
timeout := time.After(1 * time.Minute)
for {
    select {
    case <-timeout: { println("Timeout! ") }
    case resp := <-respCh: {
        fmt.Printf("Got resp: %v. ", resp)
    }
    }
}
```
```go
// 清空一个 chan
for {
    select {
    case <-ch: {}
    }
}
```
```go
// 在执行的时候可以被 done 掉
doneCh := make(chan struct{}{})

go func() {
    time.Sleep(time.Millsecond)
    close(doneCh)
}()

for {
    select {
    case <-doneCh: { return }
    default: {
        println("I'm still working... ")
    }
    }
}
```

2.  or-chan 以递归形式结合多个 chan

3.  done-chan 信道传递信息，而非共享内存传递信息

4.  pipeline 数据流水线处理
5.  fan-in/out 数据的扇入扇出

