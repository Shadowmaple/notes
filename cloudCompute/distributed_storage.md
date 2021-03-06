# 分布式存储

## CAP理论

CAP 理论（又称帽子理论）是指一个分布式系统最多只能满足**一致性（Consistency）**、**可用性（Availability）**和**分区容错性（Partition Tolerance）**中的两项。

### 一致性

一致性：在分布式的多个数据副本中保证节点数据的一致性。

+   强一致性：对于已更新过的数据，保证后续服务能够获取；
+   弱一致性：对于已更新过的数据，容忍后续的部分或全部服务无法获取；
+   最终一致性：对于已更新过的数据，经过一段时间后，最终能够被服务获取。

对于客户端来说，一致性就是在多并发访问的时候能够获取到已更新过的数据。

对于服务端来说，一致性就是将更新的数据同步到整个数据存储系统，保证数据的最终一致性。

比如节点A和节点B两个数据节点，在节点A上更新了一个数据，而要满足一致性，需要将节点A上的更新告知节点B，让节点B也同步更新。

### 可用性

可行性：数据在正常响应时间一直可用，但不保证数据是最新的副本。也就是保证客户端能够成功获取到数据，不会出现用户操作失败或访问超时的情况。

### 分区容错性

分区容错性：允许网络分区，即分布式系统在遇到网络分区节点间消息丢失或延迟时，系统能够正常提供服务。

比如节点A和节点B是网络分区的，节点A发生了故障，无法访问，但是节点B仍然能够提供服务。

### CAP 权衡

CAP在理论上只能至多满足两者，因为要考虑到系统会出现突发的情况（系统故障等）。在正常情况下，三者当然是都能满足，但一旦遇到了突发情况，则只能在三者择其二。

+   **选择CA，放弃P**：若不满足分区容错性，那么就表示只有一个网络分区，即该系统不是分布式的，放弃了系统的可扩展性。
+   **选择CP，放弃A**：选择一致性，放弃可用性，则表示当分区A的数据更新后，由于分区B的数据未更新，此时若一个客户端向分区B请求数据时，就会消息阻塞、网络延迟等，直到分区B同步了分区A的更新数据后才会重新可用。例子：传统数据库中的分布式事务ACID，需要严格保证数据一致性的场景如银行、金融、财务。
+   **选择AP，放弃C**：选择可用性，放弃一致性，则表示当分区A的数据和分区B的不一致时，对于客户端的请求，仍旧返回可用的数据，即便分区B的数据是过时的。例子：多数非关系型数据库，高流量时的12306、淘宝等。

根据CAP理论进行分布式系统设计时的注意点：

+   当存在分区时在一致性和可用性之间做出选择；
+   在特定的应用程序下，将一致性和可用性的组合利用最大化；
+   灵活地管理分区并进行分区恢复。

## 数据一致性方案

+   主从架构（Master-Slave）：一个主节点，多个从节点，一般由主节点单负责数据更新，从节点负责查询，主节点是唯一可以修改数据的成员，当从节点希望修改数据时，必须向主节点申请，主节点更新后同步到从节点。这样保证了数据强一致性。
+   多主架构（Multi-Master）：多个主节点，即多个节点可进行数据更新，以提升数据可用性和服务器响应时间。
+   两阶段提交协议（Two Phase Commit，2PC）：保证数据的强一致性。
+   三阶段提交协议（Three Phase Commit，3PC）
+   Paxos 算法
+   Raft 算法

## 文件存储



## 块存储



## 对象存储



## 分布式索引技术

+   哈希表
+   B+树
+   LSM树

## 分布式锁

+   Google Chubby
+   Zookeeper
+   阿里云女娲

## 分布式文件系统

+   GFS：Google文件系统
+   HDFS：Hadoop分布式文件系统
+   Ceph
+   Lustre
+   GlasterFS
+   阿里云盘古