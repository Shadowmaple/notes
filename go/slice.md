## slice

### slice和array

```go
runtime/slice.go
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```

slice 指向一个底层数组，不一定指向的是数组头部，也有可能是中间部分，因此多个slice可能共用一个底层数组

一般情况下，一个 slice 的 cap 取决于其底层数组的长度。如果在元素追加过程中，底层数组没有更多的空间了，那么这时候就需要申请更大的底层数组，并发生数据拷贝。这时候的 slice 的底层数组的指针地址也会发生改变。

### 扩容机制

```go
func growslice(et *_type, old slice, cap int) slice {

    if et.size == 0 {
        if cap < old.cap {
            panic(errorString("growslice: cap out of range"))
        }
        return slice{unsafe.Pointer(&zerobase), old.len, cap}
    }

    newcap := old.cap
    doublecap := newcap + newcap
    if cap > doublecap {
        newcap = cap
    } else {
        // 注意这里的 1024 阈值
        if old.len < 1024 {
            newcap = doublecap
        } else {
            // Check 0 < newcap to detect overflow
            // and prevent an infinite loop.
            for 0 < newcap && newcap < cap {
                newcap += newcap / 4
            }
            // Set newcap to the requested cap when
            // the newcap calculation overflowed.
            if newcap <= 0 {
                newcap = cap
            }
        }
    }

    var overflow bool
    var lenmem, newlenmem, capmem uintptr
    const ptrSize = unsafe.Sizeof((*byte)(nil))
    switch et.size {
    case 1:
        lenmem = uintptr(old.len)
        newlenmem = uintptr(cap)
        capmem = roundupsize(uintptr(newcap))
        overflow = uintptr(newcap) > _MaxMem
        newcap = int(capmem)
    case ptrSize:
        lenmem = uintptr(old.len) * ptrSize
        newlenmem = uintptr(cap) * ptrSize
        capmem = roundupsize(uintptr(newcap) * ptrSize)
        overflow = uintptr(newcap) > _MaxMem/ptrSize
        newcap = int(capmem / ptrSize)
    default:
        lenmem = uintptr(old.len) * et.size
        newlenmem = uintptr(cap) * et.size
        capmem = roundupsize(uintptr(newcap) * et.size)
        overflow = uintptr(newcap) > maxSliceCap(et.size)
        newcap = int(capmem / et.size)
    }

    if cap < old.cap || overflow || capmem > _MaxMem {
        panic(errorString("growslice: cap out of range"))
    }

    var p unsafe.Pointer
    if et.kind&kindNoPointers != 0 {
        p = mallocgc(capmem, nil, false)
        memmove(p, old.array, lenmem)
        // The append() that calls growslice is going to overwrite from old.len to cap (which will be the new length).
        // Only clear the part that will not be overwritten.
        memclrNoHeapPointers(add(p, newlenmem), capmem-newlenmem)
    } else {
        // Note: can't use rawmem (which avoids zeroing of memory), because then GC can scan uninitialized memory.
        p = mallocgc(capmem, et, true)
        if !writeBarrier.enabled {
            memmove(p, old.array, lenmem)
        } else {
            for i := uintptr(0); i < lenmem; i += et.size {
                typedmemmove(et, add(p, i), add(old.array, i))
            }
        }
    }

    return slice{p, old.len, newcap}
}
```

扩容时会判断 slice 的 cap 是不是已经大于 1024，如果在 1024 之内，会按二倍扩容。超过的话就是 1.25 倍扩容了。

slice 扩容必然会导致内存拷贝，如果是性能敏感的系统中，尽可能地提前分配好 slice 是较好的选择。

```go
var arr = make([]int, 0, 10)
```

### 内存分配

slice 可以用 make 和 new 创建。从表面上来说 make 会返回一个 slice 类型，而 new 则会返回一个指向 slice 的指针。

#### 源码分析

形如

```go
var a = make([]int, 10, 20)
```

的代码，会被编译器翻译为 runtime.makeslice。

```go
func makeslice(et *_type, len, cap int) slice {
    maxElements := maxSliceCap(et.size)
    if len < 0 || uintptr(len) > maxElements {
        panic(errorString("makeslice: len out of range"))
    }

    if cap < len || uintptr(cap) > maxElements {
        panic(errorString("makeslice: cap out of range"))
    }

    p := mallocgc(et.size*uintptr(cap), et, true)
    return slice{p, len, cap}
}
```

如果是

```go
var a = new([]int)
```

这样的代码，则会被翻译为:

```go
func newobject(typ *_type) unsafe.Pointer {
    return mallocgc(typ.size, typ, true)
}
```

mallocgc 函数会根据申请的内存大小，去对应的内存块链表上找合适的内存来进行分配，是 Go 自己改造的 tcmalloc 那一套。

### 值传递还是引用传递

slice 是值传递的。

从汇编层面来讲，Go 的 slice 实际上是把三个参数传到函数内部了。

所以如果在函数内对这个 slice 进行 append 时导致了 slice 的扩容，那理论上外部是不受影响的，哪怕不扩容，也可能只影响底层数组，而不影响传入的 slice。

当在另一个函数内对slice扩容时，不会改变原来的slice，如果需要改变，则需要加指针。



### 参考

-   https://github.com/cch123/golang-notes/blob/master/slice.md
-   https://qcrao91.gitbook.io/go/shu-zu-he-qie-pian/qie-pian-de-rong-liang-shi-zen-yang-zeng-chang-de

