# redis安装

# 安装

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



修改redis.conf配置文件， 改为 `daemonize yes` ，以后端模式启动，因为默认是前台启动，占用窗口

```shell
$ sudo vim /usr/local/redis/redis.confredis.conf
```



redis-benchmark   redis性能测试工具
redis-check-aof     AOF文件修复工具
redis-check-rdb     RDB文件修复工具
redis-cli      redis命令行客户端
redis.conf   redis配置文件
redis-sentinal   redis集群管理工具
redis-server  redis服务进程



## 启动与连接

启动

```shell
$ redis-server /usr/local/redis/redis.conf
$ redis-cli
```

关闭

```shell
$ redis-server shutdown
```



## 参考

https://juejin.im/post/5c3b4c4b518825253806335e