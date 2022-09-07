# Tass 思路

即Transaction As A Service；作为一个中间件，可以和多种执行服务与支持事务的存储服务相结合，以提供ACID保证的事务处理功能。

## Execution Layer

以关系型数据库为例子出发，SQL解析和执行器共同作为Execution Layer。在SQL引擎中，解析并得到执行计划，执行器按照输入的计划树执行，得到读写集。将生成读写集下发到Transaction Layer。

## Transaction Layer

Txn Node有一个线程不断收集一个epoch内接收到的事务，并为其打上唯一的tid（hash(now().timestamp, server_id)）转发掉不属于自己负责的key range的子事务。对于写操作，写buffer。epoch结束后，遍历本次epoch所有事务，根据不同的隔离级别，判断1、读集是否被改、2、写是否被其他事务覆盖这些因素，决定相应的abort结果。

__第一次同步__：**跨分片事务的原子性被破坏，各个分片无法感知子事务是否可以commit**

Region内每个节点之间同步子事务是否abort的消息。如果未收到一个节点的消息，则所有其他节点abort与该节点有关的事务。

__第二次同步：__**多主间进行CRDT合并**

每个Txn Node将经过上一步通过原子验证的写集发送到所有其他主副本。每个Txn Node必须同步收到所有的相同epoch号的所有来自其他主副本的写集，才能开始做CRDT合并。

副本间就一个epoch合并后，各个主副本对于本次所有的更新达成一致的结果（确定性合并）。

如果未收到来自某个节点的消息，

__第三次同步：由于存在确定性规则而被abort的子事务，需要再次维护事务的原子性__

副本间合并完成后，由于存在由于合并规则而abort的子事务，可能违法事务的原子性，此时需要副本间再进行一轮同步。和第一次同步应用规则一致。

__写日志__

最后写如redo log。写完log后，向Execution Layer返回commit/abort消息，同时向Storage Layer发送写请求。如果一次epoch的所有redo log均被持久化完成，则清理掉Txn Node上的缓存。

**何时返回客户端事务commit的消息？**

1、写完redo log后立刻返回：此时上一轮epoch的写还未真正持久化到Storage，存在读不到上个epoch写入的结果。如何解决？

Answer：在Txn Node确定最终的写集后，将其缓存在本地。Execution Node读取时同时读取Txn Node和Storage Node（Execution Node如何知道要读取具体哪个Node？随机读一个Node，通过集群内部转发）。如果Txn Node有数据则用，否则用Storage Node的数据。（如何清理Txn Node上的缓存，redo log 每flush一个epoch的写集，清理掉相应的缓存写集）

2、Storage Layer返回写成功后再将commit信息返回给客户端，写性能不够好。

## Storage Layer

向上提供读写的接口，支持事务的写入。

## 踩坑记录

子事务的CRDT合并不要求必须在同一个epoch号下，只要读保证一致性即可。

