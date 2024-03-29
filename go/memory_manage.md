## Golang 内存管理

>   程序中的数据和变量都会被分配到程序所在的虚拟内存中，内存空间包含两个重要区域：栈区（Stack）和堆区（Heap）。函数调用的参数、返回值以及局部变量大都会被分配到栈上，这部分内存会由编译器进行管理；不同编程语言使用不同的方法管理堆区的内存，C++ 等编程语言会由工程师主动申请和释放内存，Go 以及 Java 等编程语言会由工程师和编译器共同管理，堆中的对象由内存分配器分配并由垃圾收集器回收。

内存管理一般包含三个不同的组件，分别是用户程序（Mutator）、分配器（Allocator）和收集器（Collector），当用户程序申请内存时，它会通过内存分配器申请新内存，而分配器会负责从堆中初始化相应的内存区域。

![](https://img.draveness.me/2020-02-29-15829868066411-mutator-allocator-collector.png)

### 内存分配器

#### 设计原理

##### 分配方法 

编程语言的内存分配器一般包含两种分配方法，一种是线性分配器（Sequential Allocator，Bump Allocator），另一种是空闲链表分配器（Free-List Allocator），这两种分配方法有着不同的实现机制和特性。

###### 线性分配器

线性分配（Bump Allocator）会在内存中维护一个指向空闲区域的指针，用户程序向分配器申请内存时，分配器只需要检查剩余的空闲内存、返回分配的内存区域并修改指针在内存中的位置。

![](https://img.draveness.me/2020-02-29-15829868066435-bump-allocator.png)

优势：简单高效，执行速度快，实现复杂度低。

缺点：无法在内存被回收后重用内存；易产生内存碎片，需要额外的拼接技术或算法。



###### 空闲链表分配器

空闲链表分配器（Free-List Allocator）会在内部维护一个链表，当用户程序申请内存时，空闲链表分配器会依次遍历空闲的内存块，找到足够大的内存，然后申请新的资源并修改链表。

![空闲链表分配器](https://img.draveness.me/2020-02-29-15829868066446-free-list-allocator.png)

优点：可重用已释放的内存。

缺点：需要遍历链表，速度较慢，O(n)的时间复杂度。

空闲链表分配器可以选择不同的策略在链表中的内存块中进行选择，最常见的是以下四种：

+   首次适应（First-Fit）：从链表头开始遍历，选择第一个大小大于申请内存的内存块；
+   循环首次适应（Next-Fit）：从上次遍历的结束位置开始遍历，选择第一个大小大于申请内存的内存块；
+   最优适应（Best-Fit）：从链表头遍历整个链表，选择最合适的内存块，即大于申请内存的内存块中最小的内存块；
+   **隔离适应（Segregated-Fit）**：将内存分割成多个链表，每个链表中的内存块大小相同，申请内存时先找到满足条件的链表，再从链表中选择合适的内存块；

Golang 使用的内存分配策略类似于隔离适应，它将内存分成不同大小的内存块（4byte、8byte、16byte、32byte），不同大小的内存块由不同的链表连接：

![](https://img.draveness.me/2020-02-29-15829868066452-segregated-list.png)

使用该种分配策略，减少了需要遍历的内存块数，提高了内存分配的效率。

##### 分级分配&多级缓存

线程缓存分配（Thread-Caching Malloc，**TCMalloc**）是用于分配内存的机制。Go 语言的内存分配器就借鉴了 TCMalloc 的设计实现高速的内存分配，它的核心理念是使用**多级缓存**将对象根据大小分类，并按照类别实施不同的分配策略。

Go 语言的内存分配器会根据申请分配的内存大小选择不同的处理逻辑：

-   微对象（Tiny Object）：`(0, 16B)`
-   小对象（Small Object）：`[16B, 32KB]`
-   大对象（Large Object）：`(32KB, +∞)`

为什么要分不同对象？

因为程序中的绝大多数对象的大小都在 32KB 以下，而申请的内存大小影响 Go 语言运行时分配内存的过程和开销，所以分别处理大对象和小对象有利于提高内存分配器的性能。

分级缓存的层级：

-   **线程缓存（Thread Cache）**：属于每个独立的线程（GMP中的P），不涉及多线程竞态，所以无需锁来保护内存，减少了锁竞争带来的性能损耗。
-   **中心缓存（Central Cache）**
-   **页堆（Page Heap）**

![](https://img.draveness.me/2020-02-29-15829868066457-multi-level-cache.png)

当线程缓存不足时，会从中心缓存中补充，中心缓存不足时，会从页堆中补充。

##### 堆区域虚拟内存布局

>   在 Go 语言 1.10 以前的版本，堆区的内存空间都是连续的；但是在 1.11 版本，Go 团队使用稀疏的堆内存空间替代了连续的内存，解决了连续内存带来的限制以及在特殊场景下可能出现的问题。

线性内存

Go 语言程序的 1.10 版本在启动时会初始化整片虚拟内存区域，这些内存是虚拟内存：

![](https://img.draveness.me/2020-10-19-16031147347484/heap-before-go-1-10.png)

+   `spans` 区域：512MB，存储了指向内存管理单元 `runtime.mspan` 的指针，每个内存单元会管理几页的内存空间，每页大小为 8KB；
+   `bitmap` 区域：16GB，用于标识 `arena` 区域中的那些地址保存了对象，位图中的每个字节都会表示堆区中的 32 字节是否空闲；
+   `arena` 区域：512GB，是真正的堆区，运行时会将 8KB 看做一页，这些内存页中存储了所有在堆上初始化的对象；

稀疏内存

稀疏内存是 Go 语言在 1.11 中提出的方案，使用稀疏的内存布局不仅能移除堆大小的上限，还能解决 C 和 Go 混合使用时的地址空间冲突问题。



![](https://img.draveness.me/2020-02-29-15829868066468-heap-after-go-1-11.png)

在x86-64 的Linux系统下，整个堆区最多可以管理 256TB 的内存。



#### 内存管理组件

Go 语言的内存分配器包含以下四个重要组件：

-   内存管理单元（`runtime.mspan`）
-   线程缓存（`runtime.mcache`）
-   中心缓存（`runtime.mcentral` ）
-   页堆（`runtime.mheap`）

![](https://img.draveness.me/2020-02-29-15829868066479-go-memory-layout.png)

##### 内存管理单元

`runtime.mspan` 是 Go 语言内存管理的基本单元，该结构体中包含 next 和 prev 两个字段，它们分别指向了前一个和后一个 `runtime.mspan`，组成一个双向链表，运行时会使用 `runtime.mSpanList` 存储双向链表的头结点和尾节点并在线程缓存以及中心缓存中使用。

![](https://img.draveness.me/2020-02-29-15829868066485-mspan-and-linked-list.png)

每个 `runtime.mspan` 都管理 `npages`（mspan结构体中一个字段uintptr，表示页数） 个大小为 8KB 的页，这里的页不是操作系统中的内存页，它们是操作系统内存页的整数倍。

当用户程序或者线程向 `runtime.mspan` 申请内存时，它会使用 `allocCache` 字段以对象为单位在管理的内存中快速查找待分配的空间：

![](https://img.draveness.me/2020-02-29-15829868066499-mspan-and-objects.png)

如果能在内存中找到空闲的内存单元会直接返回，当内存中不包含空闲的内存时，上一级的组件 `runtime.mcache` 会为调用 `runtime.mcache.refill` 更新内存管理单元以满足为更多对象分配内存的需求。

`spanClass` 是 `runtime.mspan` 的跨度类，它决定了内存管理单元中存储的对象大小和个数，spanClass实际上是一个uint8 的8位整数，它的前 7 位存储着跨度类的 ID（（0-67）），最后一位表示该跨度类是否包含指针。

>   这个标识是否包含指针的位被称为 noscan 标记位，垃圾回收会对包含指针的 runtime.mspan 结构体进行扫描。比较特殊。

Go 语言的内存管理模块中一共包含 68 种跨度类，ID为1-67的每一个跨度类都会存储特定大小的对象（从8byte到32KB）并且包含特定数量的页数以及对象。其中0比较特殊，代表>32KB的跨度类。

##### 线程缓存

`runtime.mcache` 是 Go 语言中的线程缓存，它会与线程上的处理器（GMP中的P）一一绑定，主要用来缓存用户程序申请的微小对象。每一个线程缓存都持有 136 个 `runtime.mspan`，这些内存管理单元都存储在结构体的 `alloc `字段中：

>   为什么是136？
>
>   68个跨度类，每个跨度类有是否包含指针，故68*2=136

![](https://img.draveness.me/2020-02-29-15829868066512-mcache-and-mspans.png)

线程缓存在刚刚被初始化时是不包含 `runtime.mspan` 的，只有当用户程序申请内存时才会从上一级组件获取新的 `runtime.mspan` 满足内存分配的需求。

**微分配器**

线程缓存中还包含几个用于分配微对象的字段，下面的这三个字段组成了微对象分配器，专门管理 16 字节以下的非指针对象：

```go
type mcache struct {
	tiny             uintptr // 指向堆中的一片内存
	tinyoffset       uintptr // 下一个空闲内存的偏移量
	local_tinyallocs uintptr // 记录内存分配器中分配的对象个数
}
```



##### 中心缓存

```go
type mcentral struct {
	spanclass spanClass // 跨度类
	partial  [2]spanSet // 空闲对象的内存单元
	full     [2]spanSet // 已分配对象的内存单元
}
```

每个中心缓存都会管理某个跨度类的内存管理单元（所以一共有136个mcentral）

访问中心缓存需要使用互斥锁。



##### 页堆

`runtime.mheap` 是内存分配的核心结构体，Go 语言程序会将其作为全局变量存储，而堆上初始化的所有对象都由该结构体统一管理，该结构体中包含两组非常重要的字段，其中一个是全局的中心缓存列表 `central`（长度为136），另一个是管理堆区内存区域的 `arenas` 以及相关字段。



#### 内存分配

堆上所有的对象都会通过调用 `runtime.newobject` 函数分配内存，该函数会调用 `runtime.mallocgc `分配指定大小的内存空间

```go
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
	mp := acquirem()
	mp.mallocing = 1

	c := gomcache() // 获取线程缓存
	var x unsafe.Pointer
	noscan := typ == nil || typ.ptrdata == 0 // 判读是否为指针类型
    // 根据对象大小执行不同的分配逻辑
	if size <= maxSmallSize {
		if noscan && size < maxTinySize {
			// 非指针的微对象分配
		} else {
			// 小对象分配
		}
	} else {
		// 大对象分配
	}

	publicationBarrier()
	mp.mallocing = 0
	releasem(mp)

	return x
}
```

+   微对象 `(0, 16B)` ：先使用微型分配器，再依次尝试线程缓存、中心缓存和堆分配内存；
+   小对象 `[16B, 32KB]` ：依次尝试使用线程缓存、中心缓存和堆分配内存；
+   大对象 `(32KB, +∞)` ：直接在堆上分配内存；

其中<16B的指针型对象也属于小对象。

##### 微对象

Go会使用线程缓存上的**微分配器**提高微对象分配的性能，主要使用它来分配较小的字符串以及逃逸的临时变量。**微分配器可以将多个较小的内存分配请求合入同一个内存块中，只有当内存块中的所有对象都需要被回收时，整片内存才会被回收。**如下：

![](https://img.draveness.me/2020-02-29-15829868066543-tiny-allocator.png)

微分配器管理的对象不可以是指针类型，管理多个对象的内存块大小 `maxTinySize` 是可以调整的，在默认情况下，内存块的大小为 16 字节。`maxTinySize` 的值越大，组合多个对象的可能性就越高，内存浪费也就越严重；`maxTinySize` 越小，内存浪费就会越少，不过无论如何调整，8 的倍数都是一个很好的选择。

微对象分配流程：

1.  线程缓存中微分配器表示的内存块中是否有足够空闲的内存，若有，则分配返回；
2.  当内存块中不包含空闲的内存时，会先从线程缓存找到跨度类（16字节，对应跨度类为5）对应的内存管理单元 `runtime.mspan`，获取空闲的内存；
3.  当mspan不存在空闲内存时，会接着从中心缓存或者页堆中获取可分配的内存块：
4.  获取新的空闲内存块之后，会清空空闲内存中的数据并更新构成微对象分配器的字段 `tiny` 和 `tinyoffset` ；
5.  返回新的空闲内存。



##### 小对象

小对象是指>=16 字节且<= 32KB 的对象以及< 16 字节的指针类型的对象，小对象的分配可以被分成以下的三个步骤：

1.  确定分配对象的大小以及跨度类 `runtime.spanClass`；
2.  从线程缓存、中心缓存或者堆中获取内存管理单元`mspan`并从`mspan`找到空闲的内存空间；
3.  清空空闲内存中的所有数据；



##### 大对象

运行时对于大于 32KB 的大对象会单独处理，直接调用相关函数（1.14是largeAlloc）在堆上分配。

它会计算分配该对象所需要的页数，按照 8KB 的倍数在堆上申请内存。

申请内存时会创建一个跨度类为 0 的`noscan` 的`spanClass` 并调用 `runtime.mheap.alloc` 分配一个管理对应内存的管理单元`mspan`。



### 内存逃逸

golang程序变量会携带有一组校验数据，用来证明它的整个生命周期是否在运行时完全可知。如果变量通过了这些校验，它就可以在栈上分配。否则就说它逃逸了，必须在堆上分配。

能引起变量逃逸到堆上的典型情况：

+   **在方法内把局部变量指针返回** 局部变量原本应该在栈中分配，在栈中回收。但是由于返回时被外部引用，因此其生命周期大于栈，则溢出。
+   **发送指针或带有指针的值到 channel 中。** 在编译时，是没有办法知道哪个 `goroutine` 会在 `channel` 上接收数据。所以编译器没法知道变量什么时候才会被释放。
+   **在一个切片上存储指针或带指针的值。** 一个典型的例子就是 `[]*string` 。这会导致切片的内容逃逸。尽管其后面的数组可能是在栈上分配的，但其引用的值一定是在堆上。
+   **slice 的背后数组被重新分配了，因为 append 时可能会超出其容量( cap )。** slice 初始化的地方在编译时是可以知道的，它最开始会在栈上分配。如果切片背后的存储要基于运行时的数据进行扩充，就会在堆上分配。
+   **在 interface 类型上调用方法。** 在 interface 类型上调用方法都是动态调度的 —— 方法的真正实现只能在运行时知道。想像一个 io.Reader 类型的变量 r , 调用 r.Read(b) 会使得 r 的值和切片b 的背后存储都逃逸掉，所以会在堆上分配。

**是否存在逃逸，主要看变量的生命周期。**

map[int]*struct{} 呢？



### Ref

-   https://github.com/lifei6671/interview-go/blob/master/question/q019.md
-   [Go语言内存管理](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-memory-allocator/)

