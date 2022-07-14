# Highly Available Transactions: Virtues and Limitations

## HATs (VLDB2013)

## 贡献

将弱事务隔离、副本一致性和高可用系统方面的文献作了一个统一。

## High Aailability

如果用户可以联系到一个/一组正常的server(s)，则一定能受到回复。即使所有的servers存在任意的、无限长的网络分区 。

一个/一组 取决与全复制_fully replicated_ or 部分复制_partially replicated_

则称该系统提供了**高可用high availability**。

### Sticky Availability

一个弱于highly availability的概念。出现网络分区时，client的请求可能会发到不同的replica去处理。

客户端可以通过保持与服务器的stickiness来确保操作之间的连续性。

无论何时一个客户端的事务针对反映客户端所有先前操作的事务库状态副本执行时，它最终都会收到响应，则称系统提供了**sticky availability**。

### Transactional Availability

如果一个事务可以为其访问的每个item联系上至少一个replica，则称该事务具有副本可用性**replica availability**。如果给定副本可用性，该事务最终能够提交或者因内部原因abort，则称该系统提供了事务可用性**transactional availability**。如果给定**stiky availability**，事务最终能够提交或内部abort，则称系统提供了**sticky transactional availability**。

## Highly Available Transactions

---

### Achievable HAT Semantics

---

#### ACID Isolation Guarantees

写操作——$w_d(v)$: 对d项写入值v；读操作——$r_d(v)$ : 对d项读取得到v。

**Read Uncommitted**，防止“脏写”。

**Read Committed**，在RU基础上，防止“脏读”。

许多DBMS对于RR的实现有所不同。

根据ANSI标准化的，与实现无关的**Repeatable Read**定义，其防止了“Fuzzy Read” 。

为了消除“Repeatable Read”的其他定义之间的歧义，我们称呼这种property为“cut isolation”。

对于离散数据，如果可以保持这种property，则称为**Item Cut Isolation**，如果我们期望基于谓词的读取(e.g., SELECT WHERE;preventing Phantoms [38], or Berenson et al.’s P 3/A3) 我们这种更强的property为**Predicate Cut-Isolation**.

#### ACID Atomicity Guarantees

考虑原子性带来的隔离效应。定义**单调原子视图Monotonic Atomic View**。

**MAV**：一旦事务T~i~的一些effect被另一个事务T~j~观察到了，此后，所有T~i~的effect都会被T~j~观察到。

$T_1:w_x(1)w_y(1)w_z(1)$

$T_2:r_x(a)r_y(1)r_x(b)r_z(c)$

根据MAV，b=c=1，而a可以为空，也可以为1or更新的其他值。

一种实现：在RC基础上，直到所有replica都接收到了事务中的写后，才使写可见（stable）。客户端在写入时包含元数据：写时间辍以及要写入的items列表。客户端读取时，返回值的时间戳和items列表构成客户端应该为其他项目读取的版本的下限。客户端读取时，给item附带一个时间辍表示此item当前的下限。server对于每个item维护两个写入：具有最高时间辍的stable的写入，尚未stable的写入。

#### Session Guarantees

**Monotonic read**，对某个数据项的读取“从不返回任何之前的值”。换句话说，读到的值只会越来越新。

**Monotonic writes**，只有当副本包含中先前的写入时，才将写入合并到数据库副本中。

eg. 有了Monotonic Writes，保证了对文章版本N的修改会出现在版本N+1之前，否则版本N+1就会被版本N覆盖。

**Writes Follow Reads**，确保在所有服务器上的写入顺序中，保留了传统的写入/读取依赖关系。

eg. 一个写操作W~1~创建了一篇文章，R操作读取了文章内容，W~2~创建了文章的评论。那么显而易见，在其他server上，也应该先有W~1~，后有W~2~。

P186.

实现方法：server缓存写可见，直到这个写的所有的依赖在所有replica都可见。

问题：虽然ha了，但是客户端不一定能读取到自己的写入；在网络分区下，版本不会向前推进。

sticky availability提供三个额外的保证。

**Read your writes**，客户端能够读取到它写入的值或一个比它的写入更新的值。

**PRAM**（Pipelined Random Access Memory），提供了在session中序列化所有操作的illusion，并且是**Monotonic read、Monotonic write、Read your writes**的结合体。

**Causal consistency**，是上述所有session保证的组合。

### Unachievable HAT Semantics

在 HAT 系统中无法防止丢失更新或写入偏斜。无法保证recency。

### Summary

![table](../assets/HATs/HATs-table3.png)

![Figure 2](../assets/HATs/HATs-fig2.png)

## Experimental Costs

_Implementation_：一个部分复制的，基于LevelDB的原型DB。支持最终一致eventual（**last-writer-wins的RU** with **all-to-all anti-entropy between replicas**），MAV（refer to Section 5.1.2），non-HAT操作（给定key的所有操作都被路由到master执行，以及分布式2PL）

_Configuration_：以cluster为一个replica部署。（cluster类似于OB Zone的概念）cluster跨数据中心，并将数据中心内的客户端stick to相应的cluster。YCSB（0.5读0.5写），每8个YCSB操作作为一个事务。

图见论文。

图3，A&B，master的吞吐量和延迟大受影响，反之，HAT则几乎没变。当增加到5个datacenter时相同。

图4，反映了现在的MAV算法所需元数据量与数据长度成正比，消耗了IOPS和网络带宽。

图5，vary操作所占比重。

图6，扩展性，增加cluster内的server数量。