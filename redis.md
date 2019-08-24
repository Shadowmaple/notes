# Redis

标签（空格分隔）： 未分类

---

# 认识
redis是基于键值对的非关系型数据库
## 特性
1. 高性能
    + 纯内存存储，数据存放于内存中
    + 单线程架构，避免竞态和线程转换
    + I/O多路复用技术
2. 基于键值对的数据结构服务器
3. 功能丰富
4. 简单稳定
5. 客户端语言多
6. 持久化
7. 主从复制
8. 支持高可用和分布式

## redis可执行文件
| 可执行文件 | 作用 |
| :---: | :---: |
|redis-server| 启动redis
|redis-cli | redis命令行客户端
|redis-benchmark | redis基准测试工具
|redis-check-aof | redis AOF持久化文件检测和修复工具
|redis-check-dump | redis RDB持久化文件检测和修复工具
|redis-sentinel | 启动redis sentinel

# 运行
三种方法启动Redis：默认配置、运行配置、配置文件启动
```
# 关闭、启动、重启服务
redis-cli shutdown
/etc/init.d/redis-server stop
/etc/init.d/redis-server start
/etc/init.d/redis-server restart
```

```
# 查看版本
~$ redis-cli -v

# 启动客户端
~$ redis-cli
127.0.0.1:6379>
# 退出
127.0.0.1:6379> exit
```

# 操作命令
### 常用命令
```
set / mset      # (批量)设置键值对
keys *          # 查看所有键
dbsize          # 数据库内键值对个数
exists          # 键是否存在
del             # 删除键
expire          # 设置过期时间
ttl / pttl      # 查看过期时间
type            # 键值对类型
strlen          # 值长度/元素个数
get / mget / getnx / getex      # 获取键的值
incr / decr / incrby / decrby / incrbyfloat     # 值自增（整数/浮点数）
```
### 简单示例
```
# 插入键值对
127.0.0.1:6379> set python hello
OK
# 查看所有键
127.0.0.1:6379> keys *tianjia
1) "go"
2) "java"
3) "redis"
4) "python"

# 获取值
127.0.0.1:6379> get java
"no"
# 存在键
127.0.0.1:6379> exists java
(integer) 1

# 插入一个列表类型的键值对
127.0.0.1:6379> rpush mylist a b c d e f g
(integer) 7
# 查看键总数
127.0.0.1:6379> dbsize
(integer) 4
```
### 其他命令
```
append          # 字符串附加
rename          # 重命名
randomkey       # 随机抽取键


# 数据库命令
select          # 选取数据库
migrate         # 数据库迁移
flushdb / flushall  # 删除(所有)数据库
```

# 数据结构
## 字符串
字符串是redis中最基础的数据结构，键都是字符串类型。

字符串类型的值实际可以是字符串(简单的字符串、复杂的字符串(例如JSON、XML))、数字(整数、浮点数),甚至是二进制(图片、音频、视频),但值最大不能超过512MB

### 内部编码
1. **int** ：8个字节的长整型。
2. **embstr** ：小于等于39个字节的字符串。
3. **raw** ：大于39个字节的字符串。

## 哈希
在Redis中,哈希类型是指键值本身又是一个键值对结构,哈希中的映射关系叫作field-value
### 命令
哈希的全局命令基本与字符串的相同，只是在细节上以及内部实现上有一些区别
```
hset / hmset    # 设置键值对
hget / hmget    # 获取
hdel            # 删除
hlen            # 计算field个数
hexists         # 判断field是否存在
hkeys           # 获取所有field
hvals           # 获取所有value
hgetall         # 获取所有的field-value
hstrlen         # 计算value的字符串长度
```

### 内部编码
1. **ziplist(压缩列表)**：当哈希类型元素个数小于hash-max-ziplist-entries
配置(默认512个)、同时所有值都小于hash-max-ziplist-value配置(默认64
字节)时,Redis会使用ziplist作为哈希的内部实现,ziplist使用更加紧凑的
结构实现多个元素的连续存储,所以在节省内存方面比hashtable更加优秀。
2. **hashtable(哈希表)**：当哈希类型无法满足ziplist的条件时,Redis会使
用hashtable作为哈希的内部实现,因为此时ziplist的读写效率会下降,而
hashtable的读写时间复杂度为O(1)

## 列表
列表中的每个字符串称为元素(element),一个列表最多可以存储 2^32-1 个元素

特点:
有序，支持索引，支持元素重复

### 命令
| 操作类型 | 操作 |
| :---: | :---: |
| 添加 | lpush rpush linsert |
| 删除 | lpop rpop lrem ltrim |
| 查 | lrange lindex llen |
| 更改 | lset |
| 阻塞操作 | blpop brpop |

### 内部编码
1. **ziplist(压缩列表)**:当列表的元素个数小于list-max-ziplist-entries配置
(默认512个),同时列表中每个元素的值都小于list-max-ziplist-value配置时
(默认64字节),Redis会选用ziplist来作为列表的内部实现来减少内存的使
用。
2. **linkedlist(链表)**:当列表类型无法满足ziplist的条件时,Redis会使用
linkedlist作为列表的内部实现
3. quicklist: 以一个ziplist为节点的linkedlist

## 集合
特点：
无序，去重

### 命令
集合内部
```
sadd        # 添加
srem        # 删除
scard       # 计算元素个数
sismember   # 判断元素是否在集合中
srandmember # 随机返回指定个数的元素
spop        # 随机弹出元素
smembers    # 获取所有元素
```
集合间
```
sinter      # 交集
sunion      # 并集
sdiff       # 差集
sinterstore # 求交集，并将结果保存，下同
sunionstore
sdiffstore  
```

### 内部编码
1. **intset(整数集合)**:当集合中的元素都是整数且元素个数小于set-max-
intset-entries配置(默认512个)时,Redis会选用intset来作为集合的内部实
现,从而减少内存的使用。
2. **hashtable(哈希表)**:当集合类型无法满足intset的条件时,Redis会使
用hashtable作为集合的内部实现

## 有序集合
### 特点
有序，去重，通过分值实现有序，分值可重
### 操作命令
有序集合的全局命令基本与集合相同，但还是有一些变化

集合内部
```
zadd
zrem
zcard
zscore                  # 获取成员分值
zrank / zrevrank        # 计算成员排名，根据分值默认从低到高
zincrby                 # 增加成员分值
zrange / zrevrange      # 返回指定排名范围的成员
zrangebyscore / ... 
zcount                  # 返回指定分数范围成员个数
zremrangebyrank         # 删除指定排名内的升序元素
zremrangebyscore
```
集合间
```
zinterstore     # 交集
zunionstore     # 并集
```
### 内部编码
1. **ziplist(压缩列表)**:当有序集合的元素个数小于zset-max-ziplist-
entries配置(默认128个),同时每个元素的值都小于zset-max-ziplist-value配
置(默认64字节)时,Redis会用ziplist来作为有序集合的内部实现,ziplist
可以有效减少内存的使用。
2. **skiplist(跳跃表)**:当ziplist条件不满足时,有序集合会使用skiplist作
为内部实现,因为此时ziplist的读写效率会下降

# python中的redis
redis提供了很多的python语言的客户端，但最被广泛认可的是redis-py，本次便以redis-py 和 python3为例

## 准备
首先要安装redis-py库
```python
$ pip3 install redis
```

其次还要将 redis 服务器开启，开启方式可以不一样
```shell
/etc/init.d/redis-server start
```

> redis 提供 Redis 和 StrictRedis两个类，StrictRedis用于实现大部分官方的命令，并使用官方的语法和命令，Redis是StrictRedis的子类，用于向后兼容旧版本的redis-py。官方推荐使用`StrictRedis`类。

## redis连接
```python
import redis

# 连接本地的redis客户端，6479端口，默认为第一个数据库(0)，密码可设置
r = redis.StrictRedis(host = 'localhost', port = 6379, db = 0, password = None)
r.set('python', 'hello')
print(r.get('python'))

# 显示结果为 b'hello'
```

还可以用连接池 connection_pool

> redis-py使用connection pool来管理对一个redis server的所有连接，避免每次建立、释放连接的开销。默认，每个Redis实例都会维护一个自己的连接池。
可以直接建立一个连接池，然后作为参数Redis，这样就可以实现多个Redis实例共享一个连接池

```python
>>> import redis
>>>
>>> pool = redis.ConnectionPool(host='localhost', port=6379)
>>> r = redis.Redis(connetion_pool = pool)
>>> r = redis.StrictRedis(connection_pool = pool)
>>> r.set('c#', 'bu')
True
>>> r.get('c#')
b'bu'
```

在python中 redis 的应用方法基本就是一般的redis应用方式，照着用就是了

## 使用pipeline
pipeline(流水线)实际上是对一组命令进行批量操作（相当于mset,mget），即将n次客户端与redis服务端之间的往返次数（RTT）合并为一次来回，有效节约在命令传输中所损耗的时间（如网络消耗）。

```python
>>> import redis
>>> 
>>> r = redis.StrictRedis(host='localhost',port=6379)
>>> pipeline = r.pipeline()
>>> pipeline.set('vc', 2)       # 该设置并不会执行该命令，仅相当于存在一个缓冲区
Pipeline<ConnectionPool<Connection<host=localhost,port=6379,db=0>>>
>>> pipeline.incr('vc')         # 再将vc的值加一
Pipeline<ConnectionPool<Connection<host=localhost,port=6379,db=0>>>
>>>
>>> pipeline.execute()      # 执行execute后客户端才会将一组预先设定的命令传送到服务器中执行
[True, 3]
>>>
>>> r.get('vc')     # 现在可以从服务器中得到vc的值
b'3'
```


# 慢查询服务
## 慢查询
> 慢查询日志就是系统在命令执行前后计算每条命令的执行时间,当超过预设阀值,就将这条命令的相关信息(例如:发生时间、耗时、命令的详细信息)记录下来

一条Redis客户端命令的执行步骤如下：
1. 发送命令
2. 等待排队
3. 命令执行
4. 返回结果

慢查询只统计一条命令的执行所花费的时间,所以没有慢查询并不代表客户端没有超时问题

redis使用一个列表来作为慢查询日志，但却并没有暴露该列表的键，只能通过一组特定的命令来管理

## 配置
配置参数
1. slowlog-max-len : 决定慢查询日志列表存储的最大数量
2. slowlog-log-slower-than : 决定记录慢查询的时间阀值，默认单位为微秒

修改配置
1. 修改配置文件
2. config set 命令修改
```
config set slowlog-max-len 1000
config set slowlog-log-slower-than 10000

# 要Redis将配置持久化到本地配置文件，重写配置文件
config rewrite
```

## 命令
1. 获取慢查询日志
```
# 获取慢查询日志，可选参数n为数量
slowlog get [n]
```
```
127.0.0.1:6379> slowlog get
1) 1) (integer) 665
   2) (integer) 1456718400
   3) (integer) 12006
   4) 1) "SETEX"
      2) "video_info_200"
      3) "300"
      4) "2"
```
2. 获取慢查询日志列表长度
```
slowlog len
```
3. 慢查询日志重置
```
slowlog reset
```

# 零散笔记

容器型数据结构：不存在时自动创建，无元素时自动回收

过期是以对象为单位，比如一个 hash 结构的过期是整个 hash 对象的过期，而不是其中的某个子 key。

还有一个需要特别注意的地方是如果一个字符串已经设置了过期时间，然后你调用了 set 方法修改了它，它的过期时间会消失。
