## map



## sync.Map

sync.Map 能够保证并发安全的原因是维持了两个map，一个负责并发读，一个负责写，同时用互斥锁保证写操作的原子性。

结构如下：

```go
type Map struct {
	mu Mutex

    // 原子数据，实际通过类型断言转换为readOnly结构体，
    // 表示只读map数据，允许并发读
	read atomic.Value

    // 脏数据，写操作会写入该map中
	dirty map[interface{}]*entry

    // 只读map未命中但脏数据命中的次数，
    // 当到达一定次数时，会将dirty map升级为read map，
    // 下次写会创建新的dirty map
	misses int
}


type readOnly struct {
	m       map[interface{}]*entry
	amended bool // true if the dirty map contains some key not in m.
}
```



### 读数据 load

源码：

```go
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
	read, _ := m.read.Load().(readOnly)
	e, ok := read.m[key]
    // 首先从read map中读取key，
    // 若不存在并且有数据未从diry复制到read map，
    // 则尝试从diry map查找
	if !ok && read.amended {
		m.mu.Lock() // 加锁
		// Avoid reporting a spurious miss if m.dirty got promoted while we were
		// blocked on m.mu. (If further loads of the same key will not miss, it's
        // not worth copying the dirty map for this key.)
        // 再次尝试从read map查找，
        // 这样做的原因是避免在加锁期间diry map升级为read map，导致在dirty map中查找不到数据的错误未命中的情况（括号里的那句没有看懂。。）
		read, _ = m.read.Load().(readOnly)
		e, ok = read.m[key]
        // 未找到，从dirty map查找
		if !ok && read.amended {
			e, ok = m.dirty[key]
			// Regardless of whether the entry was present, record a miss: this key
			// will take the slow path until the dirty map is promoted to the read
			// map.
            // 不管该key的值是否存在，都记录一次未命中（很抱歉，后半句还是没看懂。。。）
			m.missLocked()
		}
		m.mu.Unlock() // 解锁
	}
	if !ok {
		return nil, false
	}
	return e.load()
}
```



### 总结

由于dirty map复制数据到read map中都需要时间的，且read map未找到又会到dirty map中找，所以它只适用于**读多写少**的场景，否则频繁的写入修改会导致频繁的脏数据刷新和数据miss的情况。

