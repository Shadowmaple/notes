# GMP 调度模型

## GMP 模型

实现协程并发的关键在于调度器。

GMP 模型：

-   G：goroutine
-   M：machine，指工作线程/内核线程（Thread），被 CPU 调度
-   P：processor，处理器，负责对 G 的调度，包含了运行goroutine的资源和一个可运行的 G 队列

一个 M 只有绑定了一个 P，才能运行一个 G。

除此之外，还有：

-   G队列：含有等待运行的 G，分为全局队列和P本地队列：
    -   G全局队列：访问全局G队列需要加锁
    -   P本地队列：数量有限，大小 <= 256G
-   M队列：含有处于休眠状态的 M
-   P队列：含有处于空闲状态的 P

![alt](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cfaf812fd42142e393d4b10f9859625a~tplv-k3u1fbpfcp-zoom-1.image)



设计策略：

-   **复用线程**：避免频繁的创建、销毁线程，而是对线程的复用。
    -   **work stealing**：当一个 P 的本地队列无 G 时，会先从本地队列中拿，如无，再从全局队列拿（还是相反？？）
    -   **hand off**：当本线程 M 正在调度的 G 遇到阻塞时，会释放绑定的 P，把 P 转移给其他空闲的 M 执行。
-   **利用并行**：`GOMAXPROCS`设置**P的数量**，最多有`GOMAXPROCS`个线程分布在多个CPU上同时运行。`GOMAXPROCS`也限制了并发的程度，比如`GOMAXPROCS = 核数/2`，则最多利用了一半的CPU核进行并行
-   **抢占**：在coroutine中要等待一个协程主动让出CPU才执行下一个协程，在Go中，一个goroutine最多占用CPU 10ms，防止其他goroutine被饿死，这就是goroutine不同于coroutine的一个地方。
-   **全局G队列**：在新的调度器中依然有全局G队列，但功能已经被弱化了。

## 调度机制

### 调度 G 和 Work stealing

优先从本地队列中获取G进行调度，当一个 P 的本地队列无 G 时，会先从全局队列中拿，如全局队列中无，则会启动 `work stealing`机制，尝试从其它 P 的队列中偷。（反过来？）

+   从全局队列获取：获取数量为 `min(len(GQ)/GOMAXPROCS + 1, len(GQ/2))`

+   偷的策略：获取目标队列中**后半部分的所有G**

### G 阻塞和 hand off

当本线程 M 正在调度的 G 遇到系统调用或阻塞时，会释放绑定的 P，把 P 转移给其他空闲的 M 执行。

获取空闲的M：

1.  尝试唤醒休眠的 M
2.  创建新的 M

#### 抢占机制(1.14后引入)

在 1.14 以前是协作式的，即等待协程释放才会被调度另一个协程，从1.14开始，就变为了抢占式，当一个协程占用 CPU 时间片超过10ms后，可以被其他协程抢占，被抢占后当前G暂停运行，保存当前栈的信息等待下次调度。

## 特殊的M0和G0

### M0

`M0`是启动程序后的编号为0的主线程，这个M对应的实例会在全局变量 `runtime.m0` 中，不需要在 heap 上分配，`M0`负责执行**初始化操作**和**启动第一个G**， 在之后M0就和其他的M一样了。

### G0

`G0`是每次启动一个M都会第一个创建的 gourtine，`G0`仅用于**负责调度**G，`G0`不指向任何可执行的函数, 每个M都会有一个自己的`G0`。在调度或系统调用时会使用`G0`的栈空间，全局变量的`G0`是`M0`的`G0`。

当一个G完成后，会先调度`G0`，`G0`再负责调度下一个等待调度的G。

### 自旋线程

当一个 M 和 P 正在调度 G0，且 P 本地队列为空时，处于自旋线程状态。在该状态下，会不断尝试从全局队列和其它 P 本地队列中获取可用的 G。

自旋线程数量是有限制的：自旋线程数+正在工作的线程数 = `GOMAXPROCS`

## 场景解析

### go func() 调用流程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e4a7a2193554539b2c12cc0149b798c~tplv-k3u1fbpfcp-zoom-1.image)

1.  协程创建
2.  首先会优先进入某个 P 的本地队列，如本地队列已满，则进入全局队列
3.  P 获取一个G进行调度：
    1.  本地队列有，则将队首的G进行调度
    2.  从全局队列获取
    3.  work stealing，从其它P的队列偷取
4.  M 调度 G
5.  G 执行
    1.  当 G 发生系统调用陷入内核态或发生阻塞，启动 hand off 机制，直到 G 唤醒
    2.  当 G 创建一个新的 G'
6.  销毁 G
7.  返回

### G 创建新的 G'

当一个 G 创建一个新的 G' 时，为满足局部性原理，G' 最好放在和 G 相同的P的队列中，因为可能含有相同的资源和数据。

因此：

1.  G' 会优先放于 P 的本地队列中
2.  若队列满了，则将队列分隔为前后两部分，将前部分的所有G和待放入的G1顺序打乱，再放入全局队列中，本地队列后半部分的G前移。

### 阻塞的 G 被唤醒

1.  当处于系统调用或阻塞的 G 被唤醒时，会尝试获取 P：
    1.  获取原先的那个 P
    2.  从全局的P等待队列中获取
2.  如获取不到 P，则将该 G 放入全局G队列中，等待被某个P调用

## Q&A

Q1：从其它P队列拿G，怎么保证线程安全？有锁还是怎样？

。。。

Q2：G怎么抢占的？



## Refs

+   https://mp.weixin.qq.com/s/p7sqYBUZngMfU3xXf9TNWA
+   引用最多的调度器文章：https://morsmachine.dk/go-scheduler
+   https://www.kancloud.cn/aceld/golang/1958305#99_568

