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

1.  该goroutine是队列中的最后一个对象

2.  goroutine等待时间小于1ms

与饥饿模式相比，正常模式下的互斥锁能够提供更好地性能，因为一个goroutine可以连续多次获取互斥锁，即便有阻塞的等待goroutine；而饥饿模式的能避免 goroutine 由于陷入等待无法获取锁而造成的高尾延时。

### 加锁

```go
func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
    // // 未上锁，则直接加锁成功
    // 尝试加锁（CAS无锁机制），mutexLoacked为1
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
        // 不同版本有差异，1.14.3有下面的if判断
		if race.Enabled {
			race.Acquire(unsafe.Pointer(m))
		}
		return
	}
	m.lockSlow()
}
```

如果互斥锁的状态不是 0 时就会调用 lockSlow 尝试通过自旋（Spinnig）等方式等待锁的释放，该方法的主体是一个非常大 for 循环，这里将它分成几个部分介绍获取锁的过程：

1.  判断当前 Goroutine 能否进入自旋；
2.  通过自旋等待互斥锁的释放；
3.  计算互斥锁的最新状态；
4.  更新互斥锁的状态并获取锁；



首先，判定能否自旋，若能，进入自旋，然后自旋等待互斥锁的释放：

```go
func (m *Mutex) lockSlow() {
	var waitStartTime int64
	starving := false
	awoke := false
	iter := 0
	old := m.state
	for {
		// 判断能否自旋，
        // 若是正常模式且是上锁状态的，同时系统runtime支持自旋，
        // 则可以自旋
		if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
			// 进入自旋
            // 若之前未唤醒过，且存在等待的goroutine，
            // 则将mutexWoken位设为1，即表示已有goroutine被唤醒了（实际上就是本g），
            // 这样在Unlock后就不需要再唤醒其它阻塞的goroutine了，直接释放给本g就可以了
			if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
				atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
				awoke = true
			}
			runtime_doSpin() // 进入自旋，尝试获取锁
			iter++           // 迭代次数（自旋次数）
			old = m.state
			continue
		}
        // ...
```

计算锁新的状态：

```go
		new := old
		// Don't try to acquire starving mutex, new arriving goroutines must queue.
		// 如果是饥饿模式，新的goroutine应该去排队，而不是抢锁
		if old&mutexStarving == 0 {
            // 老状态是正常状态，那么将新状态上锁
			new |= mutexLocked
		}
		// 上锁或处于饥饿状态，就将等待的g数量+1
		if old&(mutexLocked|mutexStarving) != 0 {
			new += 1 << mutexWaiterShift
		}

		// The current goroutine switches mutex to starvation mode.
		// But if the mutex is currently unlocked, don't do the switch.
		// Unlock expects that starving mutex has waiters, which will not
		// be true in this case.
		// 饥饿状态，且老状态为上锁的，则将新状态切换为饥饿模式
		if starving && old&mutexLocked != 0 {
			new |= mutexStarving
		}
		if awoke {
			// The goroutine has been woken from sleep,
			// so we need to reset the flag in either case.
			if new&mutexWoken == 0 {
				throw("sync: inconsistent mutex state")
			}
			new &^= mutexWoken
		}
		// ...
```

获取锁

```go
	// 尝试修改锁的状态
	if atomic.CompareAndSwapInt32(&m.state, old, new) {
        	// 修改成功

			if old&(mutexLocked|mutexStarving) == 0 {
				break // locked the mutex with CAS
			}
			// If we were already waiting before, queue at the front of the queue.
			queueLifo := waitStartTime != 0
			if waitStartTime == 0 {
				waitStartTime = runtime_nanotime()
			}
        	// 阻塞等待锁的信号
			runtime_SemacquireMutex(&m.sema, queueLifo, 1)
			
        	// 若等待时间超过1ms，则进入饥饿状态
			starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
			old = m.state
        	// 如果锁处于饥饿模式，那么当前goroutine直接获取锁（因为执行到这里说明它是被信号唤醒的）
        	// 若处于正常模式，会重置迭代次数，设置唤醒标识，并重新执行获取锁的循环
			if old&mutexStarving != 0 {
                // 当前goroutine被唤醒且锁处于饥饿模式
                
				// ownership was handed off to us but mutex is in somewhat
				// inconsistent state: mutexLocked is not set and we are still
				// accounted as waiter. Fix that.
                // 这时候控制权被移交到给了我们，但 mutex 不知道怎么回事处于不一致的状态:
                // mutexLocked 标识位还没有设置，但我们却仍然认为当前 goroutine 正在等待这个 mutex。说明是个 bug，需要修正
                // （有点没看懂。。。）
				if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
					throw("sync: inconsistent mutex state")
				}
				delta := int32(mutexLocked - 1<<mutexWaiterShift)
                // 如果当前goroutine不处于饥饿状态，或者它是等待的最后一个goroutine，
                // 那么就退出饥饿模式
				if !starving || old>>mutexWaiterShift == 1 {
					// Exit starvation mode.
					// Critical to do it here and consider wait time.
					// Starvation mode is so inefficient, that two goroutines
					// can go lock-step infinitely once they switch mutex
					// to starvation mode.
                    // 退出饥饿模式。
                    // 这一步骤是十分重要的，同时要考虑等待时间。
                    // 因为饥饿模式十分低效，一旦将锁切换到饥饿模式，不退出，
                    // 那么goroutines就会无限期地排队占有锁？？
                    // （这部分注释没怎么看懂。。）
					delta -= mutexStarving
				}
                // 更新锁的状态，在此表示已持有锁了，然后退出
				atomic.AddInt32(&m.state, delta)
				break
			}
			awoke = true
			iter = 0 // 重置迭代次数
		} else {
        	// 修改锁状态失败，获取锁最新的状态，重新尝试
			old = m.state
		}
	}

	if race.Enabled {
		race.Acquire(unsafe.Pointer(m))
	}
}
```





自旋是一种多线程同步机制，当前的进程在进入自旋的过程中会一直保持 CPU 的占用，持续检查某个条件是否为真。在多核的 CPU 上，自旋可以避免 Goroutine 的切换，使用恰当会对性能带来很大的增益，但是使用的不恰当就会拖慢整个程序，所以 Goroutine 进入自旋的条件非常苛刻：

1.  互斥锁只有在普通模式才能进入自旋；
2.  runtime.sync_runtime_canSpin需要返回true
    1.  运行在多 CPU 的机器上；
    2.  当前 Goroutine 为了获取该锁进入自旋的次数小于四次；
    3.  当前机器上至少存在一个正在运行的处理器 P 并且处理的运行队列为空；

一旦当前 Goroutine 能够进入自旋就会调用[`runtime.sync_runtime_doSpin` 和 `runtime.procyield` 并执行 30 次的 `PAUSE` 指令，该指令只会占用 CPU 并消耗 CPU 时间。



### 解锁



## RWMutex

