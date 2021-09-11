## Mutex

### 基本结构

go中sync.Mutex的结构：

```go
type Mutex struct {
	state int32 // 4字节的状态
	sema  uint32 // 信号
}
```

state的结构：最低三位分别表示 `mutexLocked`、`mutexWoken` 和 `mutexStarving`，剩下的位置用来表示当前有多少个 Goroutine 在等待互斥锁的释放，如图：

![golang-mutex-state](https://img.draveness.me/2020-01-23-15797104328010-golang-mutex-state.png)

-   mutexLocked：第一位，表示是否上锁
-   mutexWoken：第二位，表示是否唤醒一个goroutine
-   mutexStarving：第三位，表示是否处于饥饿状态
-   waiterCount：其余的29位表示等待的goroutine数量，最多可以表示2^29个

在源码中的定义：

```go
const (
	mutexLocked = 1 << iota // 0，是否上锁
    mutexWoken              // (10)2
    mutexStarving           // (100)2
	mutexWaiterShift = iota // 3，用于在获取等待队列数量的时候使用，作为右移的位数
)
```



### 两种模式

锁有两种模式：**正常模式和饥饿模式**

在正常状态下，会唤醒处于等待队列队首的一个goroutine，与其它新创建的goroutine同等竞争锁，但大概率会抢不到锁，因为新创建的是已经运行在CPU上的，并且数量可能有很多个（相当于共有n个，但抢到的概率只有1/n），所以会因为常常抢不到锁而饿死。

为避免饿死、保证公平，就有了饥饿状态。

饥饿状态下，所有新创建的goroutine不会去**自旋**获取锁，而是直接排到等待队列的末尾。而队首的goroutine会直接获取到释放的锁。

进入饥饿模式：

-   等待队列队首的goroutine等待时间超过**1ms**。

回到正常模式：

1.  该goroutine是等待队列中的最后一个对象

2.  该goroutine等待时间小于1ms

与饥饿模式相比，正常模式下的互斥锁能够提供更好地性能，因为一个goroutine可以连续多次获取互斥锁，即便有阻塞的等待goroutine；而饥饿模式的能避免 goroutine 由于陷入等待无法获取锁而造成的高尾延时。



### 加锁

#### 概述

一个goroutine的大致加锁过程：

1.  首先尝试CAS加锁（只有unlock且无等待的g时才会成功），不成功则进入slowPath加锁流程；
2.  能够自旋，于是尝试抢锁的woken位，并进入最多四次的自旋
    1.  获取到unlock的信号，计算锁的状态（解锁）
        1.  尝试CAS原子更新锁状态
            1.  失败：说明有其它g抢到了，重新进入自旋
            2.  成功：抢锁成功，退出
    2.  自旋次数>=4，结束自旋，去排队，计算锁的状态（等待数量+1）
        1.  尝试CAS原子更新锁状态
            1.  失败：重新for循环，计算锁的状态（因为不会重置自旋迭代次数，所以不会重新自旋）
            2.  成功：去排队，阻塞等待锁信号
                1.  如等待时间!=0，即表示是被唤醒的g，那么放入队头
                2.  如等待时间==0，即表示新到的g，需要更新它的等待时间，然后放入队尾
3.  不能自旋，要么是饥饿模式，要么runtime不允许自旋，则计算锁状态，尝试CAS原子更新状态，再放入等待队列
4.  锁从队列被唤醒：
    1.  锁处于饥饿模式：直接拿到锁，并将锁从饥饿模式退出（如果需要的话），退出
    2.  正常模式：进入for循环，CAS原子更新，抢锁，没抢到则继续进入等待队列



#### 自旋

自旋是一种多线程同步机制，当前的进程在进入自旋的过程中会一直保持 CPU 的占用，持续检查某个条件是否为真。在多核的 CPU 上，自旋可以避免 Goroutine 的切换，使用恰当会对性能带来很大的增益，但是使用的不恰当就会拖慢整个程序，所以 Goroutine 进入自旋的条件非常苛刻：

1.  互斥锁处于普通模式，同时是上锁的状态；
2.  同时满足以下条件：
    1.  运行在多核 CPU 的机器上；
    2.  当前 Goroutine 为了获取该锁进入自旋的次数小于四次；
    3.  当前机器上至少存在一个正在运行的处理器 P 并且当前P的运行队列为空；

一旦当前 Goroutine 能够进入自旋就会调用`runtime.sync_runtime_doSpin` 中的 `runtime.procyield` 并执行 30 次的 `PAUSE` 指令，该指令只会占用 CPU 并消耗 CPU 时间。



#### 源码分析

```go
func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
    // 未上锁，则直接加锁成功
    // 尝试加锁（CAS无锁机制），mutexLoacked为1
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
        // 不同版本有差异，1.14.3有下面的race判断
		if race.Enabled {
			race.Acquire(unsafe.Pointer(m))
		}
		return
	}
	m.lockSlow()
}
```

如果互斥锁的状态不是 0 时就会调用 lockSlow 尝试通过自旋（Spinnig）等方式等待锁的释放，该方法的主体是一个非常大 for 循环，大致有几个部分：

1.  判断当前 Goroutine 能否进入自旋，能则通过自旋等待互斥锁的释放；
2.  计算互斥锁的最新状态；
3.  更新互斥锁的状态并获取锁或排进队；



判定能否自旋，若能，则抢占woken位，进入自旋，等待互斥锁的释放：

```go
func (m *Mutex) lockSlow() {
	var waitStartTime int64
	starving := false  // 当前g是否处于饥饿状态
	awoke := false  // awoke表示锁的唤醒位是因为当前g被唤醒而被设置的
	iter := 0
	old := m.state
	for {
		// 判断能否自旋，进入自旋：
        // 正常模式且是上锁状态的，同时runtime支持自旋
		if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
			// 进入自旋
            // 若之前未唤醒过，且存在等待的goroutine，
            // 则将mutexWoken位设为1，即表示已有goroutine在等待unlock了（实际上就是本g），
            // 这样在Unlock后就不需要再唤醒其它睡眠的g了，以免唤起太多的g
			if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
				atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
				awoke = true
			}
			runtime_doSpin() // 进入自旋，尝试获取锁
			iter++           // 迭代次数（自旋次数）
			old = m.state
			continue
		}
        // 自旋次数大于4，或已解锁，或饥饿模式，则尝试去修改锁的状态、获取锁
        // ...
```

计算锁新的状态：

1.  设置lock加锁位：若是正常模式，则加锁；
2.  等待数量+1：如果是locked或饥饿模式，那么就表示需要排队；
3.  设置饥饿模式：若g处于饥饿状态，且是locked的，则设置饥饿模式；
4.  重置mutexWoken位：如果当前g持有唤醒标识（或者说是被唤醒的），则重置；

```go
		new := old
		// 如果是饥饿模式，新的goroutine应该去排队，而不是抢锁

		// 若是正常模式，则要设置加锁。
		// 为什么要有这个？若是正常模式，进入到这里，说明要么等到了互斥锁释放，要么不能自旋了
		// 若是等待了互斥锁释放，那应该去尝试加锁；
		// 若不能自旋，那表示锁已经是锁上的状态，再置位区别不大
		if old&mutexStarving == 0 {
			new |= mutexLocked
		}

		// 上锁或处于饥饿模式，就将等待的g数量+1
		// 若是饥饿模式，说明是新到达的g，应该直接去排队（唤醒的呢？）
		// 若是上锁，到达这里说明不能自旋了，也应该去排队，不能空耗CPU
		if old&(mutexLocked|mutexStarving) != 0 {
			new += 1 << mutexWaiterShift
		}

		// The current goroutine switches mutex to starvation mode.
		// But if the mutex is currently unlocked, don't do the switch.
		// Unlock expects that starving mutex has waiters, which will not
		// be true in this case.
		// g处于饥饿状态，且锁住的，则切换为饥饿模式。
		// g处于饥饿状态说明是被唤醒的，且for循环迭代了不只一次，因为这个starving变量是后面才更新的
		// 为什么unlock时不切换饥饿状态？如果unlock了，从性能考虑，再去抢一次更好，不然只能去排队了
		// unlock时会希望有等待的goroutine？因为没有等待unlock的g就需要去唤醒一个g
		if starving && old&mutexLocked != 0 {
			new |= mutexStarving
		}

		// 如果持有唤醒标识的话（或者说该goroutine被唤醒），需要将锁的woken位重置。
		// 因为如果之后CAS原子操作成功，无论是抢到了锁还是排进了队，当前g都不是等待unlock的唤醒的g了
		if awoke {
			if new&mutexWoken == 0 {
				throw("sync: inconsistent mutex state")
			}
            // 与+异或（AND NOT）=位清空，将1位置0
            // 将mutexWoken位置为0
			new &^= mutexWoken
		}
		// ...
```

修改锁的状态并获取锁：

1.  若果CAS原子操作修改锁状态成功：
    1.  抢锁成功（锁是正常模式且是unlock的），退出
    2.  进入等待队列排队，阻塞等待锁的信号：
        1.  如果是之前被唤醒过的（等待时间!=0），则排入队头
        2.  如是第一次进入的（等待时间==0），则更新等待时间，并排入队尾
2.  CAS原子操作失败：更新锁最新的状态，再次自旋尝试，等待

```go
	// 尝试修改锁的状态
	// 若失败，则重新获取锁的状态，重新尝试。因为说明有其它g抢先更新了，现在的new是失效的，要重新计算
	if atomic.CompareAndSwapInt32(&m.state, old, new) {
        	// 修改成功

        	// 若状态是unlock且正常模式，则判定获取锁成功，直接退出
        	// 如果是饥饿模式，新来的g就去排队吧！运行到这里的不可能是饥饿模式下被唤醒的g
        	// 如果是lock的，说明到这里的g是因为无法自旋只能排队了
			if old&(mutexLocked|mutexStarving) == 0 {
				break // locked the mutex with CAS
			}

			// If we were already waiting before, queue at the front of the queue.
        	// 如果之前等待过了，说明是从队列中唤醒的，重新放到队头Last in first out
			queueLifo := waitStartTime != 0
        	// 为新创建的g，更新当前的时间
			if waitStartTime == 0 {
				waitStartTime = runtime_nanotime()
			}
        	// 阻塞等待锁的信号
			runtime_SemacquireMutex(&m.sema, queueLifo, 1)

        	// 被唤醒
        	// 若等待时间超过1ms，则g进入饥饿状态
			starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
			old = m.state
        	// 如果锁处于饥饿模式，那么当前被唤醒的g会直接获取锁
        	// 若处于正常模式，则会重置迭代次数，设置唤醒标识，并重新执行获取锁的循环
			if old&mutexStarving != 0 {
                // 当前goroutine被唤醒且互斥锁处于饥饿模式

                // 这时候控制权被移交到给了我们，但 mutex 不知道怎么回事处于不一致的状态:
                // mutexLocked 标识位还没有设置，但我们却仍然认为当前 goroutine 正在等待这个 mutex。说明是个 bug，需要修正
                // 状态是上锁或是woken为1，或者没有等待的g，则报错：
                // 1.到达这里说明是unlock的，不然不会唤醒一个g
                // 2.woken为1报错？unlock时会唤醒一个g执行到这里说明锁的woken位是0
                // 3.没有等待的g就不会是饥饿模式了，也不会有该被唤醒的g
				if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
					throw("sync: inconsistent mutex state")
				}
                // 先左移，再减
                // （delta没怎么看懂）
				delta := int32(mutexLocked - 1<<mutexWaiterShift)
                // 若当前goroutine不处于饥饿状态，或它是最后一个等待者，那么锁就退出饥饿模式。
                // 这一步骤是十分重要的，同时要考虑等待时间，
                // 因为饥饿模式十分低效，一旦将锁切换到饥饿模式，不退出，
                // 那么goroutines就会无限期地排队等待锁，十分损耗性能。
				if !starving || old>>mutexWaiterShift == 1 {
					delta -= mutexStarving
				}
                // 更新锁的状态，在此表示已持有锁了，然后退出
                // （没有看懂这个操作，delta？？）
				atomic.AddInt32(&m.state, delta)
				break
			}
			awoke = true
			iter = 0 // 重置自旋次数
		} else {
			old = m.state
		}
	}

	if race.Enabled {
		race.Acquire(unsafe.Pointer(m))
	}
}
```





### 解锁

#### 概述

一个Goroutine unlock的大体流程：

1.  首先原子操作解锁，locked 位-1
2.  判断状态是否为0，为0说明没有其它在等待的g，直接退出
3.  若有其它等待的g，则需要尝试唤醒
    1.  正常模式：通过判断等待数量、locked位、woken位和starving位，是否需要唤醒一个g，需要则尝试CAS获取唤醒的权力，唤醒
    2.  饥饿模式：直接唤醒等待队头的g



#### 源码分析

```go
// 锁不会与特定的goroutine关联，
// 即允许一个协程加锁，但由另一个协程解锁
func (m *Mutex) Unlock() {
	// ...race的相关判断

	// Fast path: drop lock bit.
    // 对没有等待队列的锁直接解锁
    // 若new!=0，说明存在等待加锁的协程，需要调用unlockSlow处理一些逻辑
	new := atomic.AddInt32(&m.state, -mutexLocked)
	if new != 0 {
		// Outlined slow path to allow inlining the fast path.
		// To hide unlockSlow during tracing we skip one extra frame when tracing GoUnblock.
		m.unlockSlow(new)
	}
}
```

unlockSlow的逻辑：

```go
func (m *Mutex) unlockSlow(new int32) {
    // 此时new已经-1，即为解锁后的值，正常情况下最低位mutexLocked为0
    // 若+1后为0，则原本为0，即unlocked，即解了一个未加锁的锁，属于运行时错误
	if (new+mutexLocked)&mutexLocked == 0 {
		throw("sync: unlock of unlocked mutex")
	}

    // 若是正常模式，如果需要唤醒某个g，则尝试去唤醒
    // 若是饥饿模式，释放信号给等待的队头g，不用设置唤醒标识位
	if new&mutexStarving == 0 {
		old := new
		for {
			// If there are no waiters or a goroutine has already
			// been woken or grabbed the lock, no need to wake anyone.
			// In starvation mode ownership is directly handed off from unlocking
			// goroutine to the next waiter. We are not part of this chain,
			// since we did not observe mutexStarving when we unlocked the mutex above.
			// So get off the way.
            // 如果没有等待的g，或locked的（已经有其它g上锁了）、唤醒了的（有其它g在等待）、或饥饿模式，则直接return，不用再唤醒一个g了
			if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
				return
			}
            // 获取唤醒的权利。
            // 等待数-1，将设置唤醒位，并使用CAS更新状态，失败则重复尝试
			new = (old - 1<<mutexWaiterShift) | mutexWoken
			if atomic.CompareAndSwapInt32(&m.state, old, new) {
                // 释放信号，唤醒一个g
				runtime_Semrelease(&m.sema, false, 1)
				return
			}
			old = m.state
		}
	} else {
		runtime_Semrelease(&m.sema, true, 1)
	}
}
```



### semaphore



## RWMutex

读写锁的实现类似于操作系统的PV操作的信号量机制，读者写者问题、生产者消费者问题……



### 读锁



### 写锁