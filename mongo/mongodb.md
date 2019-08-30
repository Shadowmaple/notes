# mongodb
## 安装
>    [官网下载](https://www.mongodb.com/download-center/community?jmp=docs)

```shell
_ tar -xzf 下载/mongodb-linux-x86_64-ubuntu1804-4.2.0.tgz -C /usr/local/
_ mv /usr/local/mongodb-linux-x86_64-ubuntu1804-4.2.0 /usr/local/mongodb
mkdir-p  data/db
mkdir log
```
建立 `mongodb.conf` 配置文件
```shell
dbpath = /usr/local/mongodb/data/db
logpath = /usr/local/mongodb/log/mongodb.log
port = 27017
# 后台运行
fork = true
logappend = true
```

## 账户
账户
admin数据库：admin mongoadmin
table数据库：admin admin

## 连接启动服务
```shell
# 当前目录为/usr/local/mongodb
# 启动服务
$ ./bin/mongod --conf mongodb.conf 
# 或者mongod -f mongodb.conf ，mongo已添加到path中

# 连接mongo
$ ./bin/mongo
# mongo

# 停止服务
$ ./bin/mongod -shutdown -f mongodb.conf
# mongod -shutdown -f mongodb.conf
```
注意：如果这样启动不了，试试加sudo。如果这样可以启动，就把文件权限者更改为非roo用户

## 操作
```
show dbs
db.foo.find()
show collections
db.table_xk.find()
```

## 创建用户
```shell
db.createUser({
   user: "admin",
   pwd: "admin",
   roles: [ { role: "dbOwner", db: "ccnubox"} ]
})
```

## 权限角色
### 数据库用户角色
read: 只读数据权限
readWrite:学些数据权限

### 数据库管理角色
dbAdmin: 在当前db中执行管理操作的权限
dbOwner: 在当前db中执行任意操作
userADmin: 在当前db中管理user的权限

### 备份和还原角色
backup
restore

### 夸库角色
readAnyDatabase: 在所有数据库上都有读取数据的权限
readWriteAnyDatabase: 在所有数据库上都有读写数据的权限
userAdminAnyDatabase: 在所有数据库上都有管理user的权限
dbAdminAnyDatabase: 管理所有数据库的权限

### 集群管理
clusterAdmin: 管理机器的最高权限
clusterManager: 管理和监控集群的权限
clusterMonitor: 监控集群的权限
hostManager: 管理Server

### 超级权限
root: 超级用户