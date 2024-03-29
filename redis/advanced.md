# Redis

## 线程IO模型

redis是单线程的工作模式，但是仍有其它线程帮助处理IO、异步等事件。

redis采用IO多路复用，采用select、expoll等系统调用绑定多个IO事件，read或write完成就进行通知。

大致伪代码如下：

```python
read_events, write_events = select(read_fds, write_fds, timeout)
for event in read_events:
	handle_read(event.fd)
for event in write_events:
	handle_write(event.fd)
handle_others() # 处理其它事情，如定时任务等
```

**指令队列**：Redis 会将每个客户端套接字都关联一个指令队列。客户端的指令通过队列来排队进行顺序处理，先到先服务。  

**响应队列**：同样也会为每个客户端套接字关联一个响应队列。Redis 服务器通过响应队列来将
指令的返回结果回复给客户端。 如果队列为空，那么意味着连接暂时处于空闲状态，不需要
去获取写事件，也就是可以将当前的客户端描述符从 write_fds 里面移出来。等到队列有数据
了，再将描述符放进去。避免 select 系统调用立即返回写事件，结果发现没什么数据可以
写。出这种情况的线程会飙高 CPU 。

**定时任务**：Redis 的定时任务会记录在一个**最小堆**的数据结构中。这个堆中，最快要执行的任务排在堆的最上方。在每个循环周期， Redis 都会将最小堆里面已经到点的任务立即进行处理。处理完毕后，将最快要执行的任务还需要的时间记录下来，这个时间就是 select 系统调用的 timeout 参数。  有了这个 timeout，Redis 就不会阻塞在 IO 事件上，可以处理其它任务，比如定时任务。



## 事务

使用：

- multi：事务的开始
- exec：事务的执行
- discard：事务的丢弃

Redis 的事务在实现上是将所有命令事先缓存到一个事务队列中，直到触发 exec 指令才一起发到工作线程执行。所以很好的实现了事务隔离性中的串行化条件。但是并没有达到原子性。因为在执行中若有某个命令出现错误，那么后续的命令仍旧会继续执行，redis 不提供回滚操作。

可以采用 watch 命令，来监视某个 Redis 对象，若该对象在事务执行前被修改了，即不是期望的值，那么该事务就会宣告失败。（乐观锁的形式）

虽然事务的指令是集中一起执行的，但是每条命令的发送都占用一次网络IO，所以一般会采用 pipline 将多次IO合并成一次IO。



Q：Redis 为什么不提供回滚操作？



## 过期策略

Redis 的过期策略：

1. 惰性删除：下一次请求该key时先检查是否过期，若过期则删除；
2. 定时扫描：redis 会将每个设置了过期时间的 key 放入到一个独立的字典中，默认**每秒十次**进行过期扫描，不会遍历所有key，会随机选取扫描。

定时扫描策略：

1. 从过期字典中随机 20 个 key；
2. 删除这 20 个 key 中已经过期的 key；
3. 如果过期的 key 比率超过 1/4， 那就重复步骤 1；  

同时，为了保证过期扫描不会出现循环过度，导致线程卡死现象，算法还增加了扫描时
间的上限，默认不会超过 25ms。

定时扫描、删除过期的key也会占用线程的处理时间，如果太过繁忙，会造成客户端的阻塞、卡顿，同时内存管理器需要频繁回收内存页，这也会产生一定的 CPU 消耗。



从库的过期策略：

- 从库不会进行过期扫描，从库对过期的处理是被动的。主库在 key 到期时，会在 AOF文件里增加一条 del 指令，同步到所有的从库，从库通过执行这条 del 指令来删除过期的key。

- 因为指令同步是异步进行的，所以主库过期的 key 的 del 指令没有及时同步到从库的话，会出现主从数据的不一致，主库没有的数据在从库里还存在  



## 异步删除-懒处理

对于del删除一些大对象（比如大的哈希表），会比较耗时，Redis 会将该操作丢给后台线程异步处理。

Redis 4.0 引入 `unlink` 指令，对删除操作进行懒处理，丢给后台线程来异步回收内存。

在使用的使用，一开始就会把对象的引用给去除了，所以不会产生多线程并发问题。

> 比较形象的比喻：删除的对象是枝干，unlink 会将枝干从大树上去除，这样大树就无法访问到枝干，不会产生并发问题，枝干的回收异步进行。



对于 `flushdb`和`flushall`两个清空数据库的指令，Redis4.0也提供一个 `async` 指令来异步处理。

```redis
flushall async
```



主线程将对象的引用从「大树」中摘除后，会将这个 key 的内存回收操作包装成一个任务，塞进异步任务队列，后台线程会从这个异步队列中取任务。任务队列被主线程和异步线程同时操作，所以必须是一个线程安全的队列。  



Redis 回收内存除了 del 指令和 flush 之外，还会存在于在 key 的过期、 LRU 淘汰、rename 指令以及从库全量同步时接受完 rdb 文件后会立即进行的 flush 操作。Redis4.0 为这些删除点也带来了异步删除机制。



## 管道 pipline

管道pipline 实际上不是 Redis 服务器提供的一种特别的技术，而是 Redis 客户端提供的。

本质来说，就是 Redis 客户端将一系列指令写入操作系统的内核缓冲区 send buffer 中，然后进行一次网络IO发送给 Redis 服务器。相比较未使用管道，它将多次发送指令的网络IO合并为一次。当然，发送过后还有 read 需要等待响应，一次网络IO。

本质上是通过改变读写顺序而带来的性能提升。

