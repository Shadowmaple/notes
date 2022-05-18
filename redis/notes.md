# Redis

## 初识

Redis 是基于键值对的内存型非关系型数据库



特性：

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



## 安装

官网下载最新的stable安装包

> https://redis.io/

解压到指定目录

```shell
$ tar -xzvf Downloads/redis-5.0.5.tar.gz -C software/
```

用 `make test` 命令测试一下。

可能出现提示 `You need tcl 8.5 or newer in order to run the Redis test` ，这是缺少 tcl 包，安装一下 tcl 就好了（如 `apt-get install tcl`)

测试完成，就可以安装 Redis 了，先 cd 到 Redis 解压文件的 src 目录，使用 `make PREFIX=/usr/local/redis install` 安装，可以设置 Redis 的安装位置

复制配置文件到安装目录

```shell
$ sudo cp software/redis-5.0.5/redis.conf /usr/local/redis/
```



将redis的可执行文件添加到PATH中，打开 `.zshrc` 文件，添加

```shell
# redis
export PATH=$PATH:/usr/local/redis/bin
```

再source 一下

```shell
$ source .zshrc
```



修改`redis.conf`配置文件， 改为 `daemonize yes` ，以后端模式启动，因为默认是前台启动，会占用前台窗口。

```shell
$ sudo vim /usr/local/redis/redis.confredis.conf
```



## 可执行文件

| 可执行文件      | 作用                              |
| --------------- | --------------------------------- |
| redis-server    | redis服务器                       |
| redis-cli       | redis命令行客户端                 |
| redis-benchmark | redis基准测试工具                 |
| redis-check-aof | redis AOF持久化文件检测和修复工具 |
| redis-check-rdb | redis RDB持久化文件检测和修复工具 |
| redis-sentinel  | 启动redis sentinel                |
| redis.conf      | 配置文件                          |



## 启动与运行

Redis 属于C/S模式，由客户端向服务端发起请求。

### Redis 服务器

三种方法启动Redis：默认配置、运行配置、配置文件启动

```shell
# 服务启动、停止、重启、关闭
$ redis-server start
$ redis-server stop
$ redis-server restart
$ redis-server shutdown

# 以某配置文件启动
$ redis-server /usr/local/redis/redis.conf
```



### Redis 客户端

```shell
# 查看版本
~$ redis-cli -v

# 启动客户端，连接
~$ redis-cli
127.0.0.1:6379>
# 退出
127.0.0.1:6379> exit
```



## 操作指令

### 常用命令

```shell
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

```shell
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

```shell
append          # 字符串附加
rename          # 重命名
randomkey       # 随机抽取键

# 数据库命令
select          # 选取数据库
migrate         # 数据库迁移
flushdb / flushall  # 删除(所有)数据库
```



## Python中使用

redis提供了很多的python语言的客户端，但最被广泛认可的是redis-py，本次便以redis-py 和 python3为例

### 准备

首先要安装redis-py库

```shell
$ pip3 install redis
```

其次还要将 redis 服务器开启，开启方式可以不一样

```shell
redis-server start
```

> redis 提供 Redis 和 StrictRedis两个类，StrictRedis用于实现大部分官方的命令，并使用官方的语法和命令，Redis是StrictRedis的子类，用于向后兼容旧版本的redis-py。官方推荐使用`StrictRedis`类。

### redis连接

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
> 可以直接建立一个连接池，然后作为参数Redis，这样就可以实现多个Redis实例共享一个连接池

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

### 使用pipeline

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



## 慢查询服务

### 慢查询

> 慢查询日志就是系统在命令执行前后计算每条命令的执行时间,当超过预设阀值,就将这条命令的相关信息(例如:发生时间、耗时、命令的详细信息)记录下来

一条Redis客户端命令的执行步骤如下：

1. 发送命令
2. 等待排队
3. 命令执行
4. 返回结果

慢查询只统计一条命令的执行所花费的时间,所以没有慢查询并不代表客户端没有超时问题

redis使用一个列表来作为慢查询日志，但却并没有暴露该列表的键，只能通过一组特定的命令来管理

### 配置

配置参数

1. slowlog-max-len : 决定慢查询日志列表存储的最大数量
2. slowlog-log-slower-than : 决定记录慢查询的时间阀值，默认单位为微秒

修改配置

1. 修改配置文件
2. config set 命令修改

```shell
config set slowlog-max-len 1000
config set slowlog-log-slower-than 10000

# 要Redis将配置持久化到本地配置文件，重写配置文件
config rewrite
```

### 命令

1. 获取慢查询日志

```shell
# 获取慢查询日志，可选参数n为数量
slowlog get [n]
```

```shell
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

```shell
slowlog len
```

3. 慢查询日志重置

```shell
slowlog reset
```



## 参考

- https://juejin.im/post/5c3b4c4b518825253806335e