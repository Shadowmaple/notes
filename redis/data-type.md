# Redis数据结构

Redis 有五大基本数据类型：字符串、哈希表、列表、集合和有序集合。

![数据结构](https://images2017.cnblogs.com/blog/1260387/201712/1260387-20171217225104530-830166094.png)

## 字符串
字符串是redis中最基础的数据结构，键都是字符串类型。

字符串类型的值实际可以是字符串(简单的字符串、复杂的字符串(例如JSON、XML))、数字(整数、浮点数),甚至是二进制(图片、音频、视频)，但值最大不能超过512MB

### 内部编码
1. **int** ：8个字节的长整型。
2. **embstr** ：小于等于39个字节的字符串。
3. **raw** ：大于39个字节的字符串。

## 哈希
在Redis中，哈希类型是指键值本身又是一个键值对结构，哈希中的映射关系叫作field-value
### 命令
哈希的全局命令基本与字符串的相同，只是在细节上以及内部实现上有一些区别
```shell
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
1. **ziplist(压缩列表)**：当哈希类型元素个数小于`hash-max-ziplist-entries` 配置(默认512个)、同时所有值都小于`hash-max-ziplist-value`配置(默认64字节)时，Redis会使用 ziplist 作为哈希的内部实现，ziplist使用更加紧凑的结构实现多个元素的连续存储，所以在**节省内存**方面比hashtable更加优秀。
2. **hashtable(哈希表)**：当哈希类型无法满足ziplist的条件时，Redis会使用hashtable作为哈希的内部实现，因为此时ziplist的读写效率会下降，而hashtable的读写时间复杂度为O(1)

## 列表
列表中的每个字符串称为元素(element)，一个列表最多可以存储 `2^32-1` 个元素

特点：有序，支持索引，支持元素重复

### 命令
| 操作类型 | 操作 |
| :---: | :---: |
| 添加 | lpush rpush linsert |
| 删除 | lpop rpop lrem ltrim |
| 查 | lrange lindex llen |
| 更改 | lset |
| 阻塞操作 | blpop brpop |

### 内部编码
1. **ziplist(压缩列表)**：当列表的元素个数小于`list-max-ziplist-entries`配置(默认512个)，同时列表中每个元素的值都小于`list-max-ziplist-value`配置时(默认64字节)，Redis会选用ziplist来作为列表的内部实现来减少内存的使用。
2. **linkedlist(链表)**：当列表类型无法满足ziplist的条件时，Redis会使用linkedlist作为列表的内部实现
3. **quicklist**: 以一个ziplist为节点的linkedlist

## 集合
特点：无序，去重

### 命令
集合内部
```shell
sadd        # 添加
srem        # 删除
scard       # 计算元素个数
sismember   # 判断元素是否在集合中
srandmember # 随机返回指定个数的元素
spop        # 随机弹出元素
smembers    # 获取所有元素
```
集合间
```shell
sinter      # 交集
sunion      # 并集
sdiff       # 差集
sinterstore # 求交集，并将结果保存，下同
sunionstore
sdiffstore  
```

### 内部编码
1. **intset(整数集合)**：当集合中的元素都是整数且元素个数小于`set-max-intset-entries`配置(默认512个)时，Redis会选用 intset 来作为集合的内部实现，从而减少内存的使用。
2. **hashtable(哈希表)**：当集合类型无法满足intset的条件时，Redis会使用hashtable作为集合的内部实现

## 有序集合

特点：有序，去重，通过分值实现有序，分值可重

### 操作命令
有序集合的全局命令基本与集合相同，但还是有一些变化

集合内部
```shell
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
```shell
zinterstore     # 交集
zunionstore     # 并集
```
### 内部编码
1. **ziplist(压缩列表)**：当有序集合的元素个数小于`zset-max-ziplist-entries`配置(默认128个)，同时每个元素的值都小于`zset-max-ziplist-value`配置(默认64字节)时，Redis会用ziplist来作为有序集合的内部实现，ziplist可以有效减少内存的使用。
2. **skiplist(跳跃表)**：当ziplist条件不满足时，有序集合会使用skiplist作为内部实现，因为此时ziplist的读写效率会下降



## 其它

容器型数据结构：不存在时自动创建，无元素时自动回收

过期是以对象为单位，比如一个 hash 结构的过期是整个 hash 对象的过期，而不是其中的某个子 key。

还有一个需要特别注意的地方是如果一个字符串已经设置了过期时间，然后你调用了 set 方法修改了它，它的过期时间会消失。

