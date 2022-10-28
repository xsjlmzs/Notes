# Cassandra

## ABSTRACT

本文介绍了Dynamo的设计和实现。

## 1. INTRODUCTION

．．．

## 2. RELATEＤ　WORK

．．．

## 3. DATA MODEL

Cassandra 中的表是由键索引的分布式多维映射。 值是高度结构化的对象。无论读取或写入多少列，单个行键下的每个操作都是每个副本的原子操作。和BigTable一样，列被分到列族中。此外，有超级列族的概念：可以看作在列族里的列族。

应用程序可以指定超级列族或简单列族中列的排序顺序（按时间or名称）。列族中的列通过_column_family:column_访问。超级列族中的列通过_column_family:super_column:column_访问。

## 4. API

Cassandra API包含下面三个：

- insert(table, key, rowMutation)
- get(table, key, columnName)
- delete(table, key, columnName)

columnName可以是一个列族中指定的列、一个列族、一个超级列族或者一个超级列族中的列。

## 5. SYSTEM ARCHITECTURE

对于写入，系统将请求路由到副本并等待一定数量的副本确认写入完成。对于读取，根据客户端要求的一致性保证，系统要么将请求路由到最近的副本，要么将请求路由到所有副本并等待一定数目的响应。

### 5.1 Partitioning

Cassandra使用一致性哈希对集群中的数据进行分区。对key进行哈希，然后按顺时针遇到的第一个节点作为该key的协调节点。每个节点都负责环中它与其在环上的前任节点之间的区域。

对一致性哈希的不足（节点的负载分布不均，忽略了节点性能的差异）进行了改进：分析环上的信息，让负载较轻的节点移动以平衡负载，如[17]。

### 5.2 Replication

和Dynamo一样，每个数据项在 N 个主机上复制。每个key分配给一个协调节点，协调节点负责key的复制。Cassandra提供了多种复制策略，如“Rack Unaware”、“Rack Aware”（在数据中心内）和“Datacenter Aware”。如果选择“Rack Unaware”，则通过选择环上协调器的N-1个后继来选复制。

使用Zookeeper选举出一个leader，所有加入集群的节点都要联系leader，leader告诉它们作为哪个range的副本存在。

领导者齐心协力维护不变量，即没有节点负责环中超过 N-1 个范围。 有关节点负责范围的元数据在每个节点本地缓存，并在 Zookeeper 内部以容错方式缓存

### 5.3 Membership

Cassandra中，Gossip不仅用于成员资格，还用于传播其他系统相关的控制状态。

#### 5.3.1 Failure Detection

使用Φ Accrual Failure Detector[8]的一种修改版本：不是使用bool值来说明节点是启动还是关闭，而是使用一个值Φ来表示每个受监控节点的怀疑程度。

具体如下：假设当 Φ = 1 时我们决定怀疑节点 A，那么我们将犯错误的可能性（即，该决定在将来会因接收到心跳延迟）约为 10%。 Φ = 2 的可能性约为 1%，Φ = 3 的可能性约为 0.1%，依此类推。系统中的每个节点都维护着一个滑动窗口，该窗口是来自集群中其他节点的gossip消息的到达时间间隔。确定这些到达时间间隔的分布并计算Φ。

### 5.4 Bootstrapping

节点第一次启动时，会在环上随机选择一个位置，然后将该信息持久化到本地硬盘和Zookeeper中。然后通过gossip在集群中传播。

和Dynamo一样，需要一些种子节点在集群中以便新加入的节点联系。

### 5.5 Scaling the Cluster

### 5.6 Local Persistence

WAL,先写提交日志再执行内存的写入。当内存中的数据超过某个阈值，就转储到磁盘。按顺序写入磁盘并根据行键生成索引以进行有效的查找。随着时间文件数目会愈来愈多，类似BigTable的压缩过程一般进行文件合并。

查询时，先查内存再查磁盘。查找可能会在多个文件中查找一个key，所以每个文件使用一个布隆过滤器汇总文件中的key并保存在内存中。

维护列索引以快速跳转到磁盘上的正确块进行查找。

### 5.7 Implementation Details

SEDA架构。所有系统控制消息都依赖于基于 UDP 的消息传递，而用于复制和请求路由的应用程序相关消息依赖于 TCP。

滚动提交日志。每次将特定列族的内存数据结构转储到磁盘时，我们都会在提交日志中设置它的位，说明该列族已成功保存到磁盘。每次滚动提交日志时，都会检查其位向量以及在其之前滚动的提交日志的所有位向量。如果认为所有数据都已成功持久化到磁盘，则删除这些提交日志。

## 6. PRACTICAL EXPERIENCES

#### 6.1 Facebook Index Search

## 7. CONCLUSION

Cassandra 可以支持非常高的更新吞吐量，同时提供低延迟。 未来的工作包括添加压缩、支持跨key的原子性和支持二级索引的能力。