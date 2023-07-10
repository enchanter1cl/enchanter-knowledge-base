

> 节选和讲解《DDIA》（数据密集型应用的设计）的第 5 章，是写的非常好的一章节



##  引言 - 分布式

你可能会出于各种各样的原因，希望将数据分布到多台机器上：为了 **容错/高可用性 and 低延迟**。

[vertical-scaling/scaling up]

【纵向延伸还是水平扩展】

纵向延伸即提升单机性能，比如超级计算机。这样毕竟容错能力有限。虽然高端机器可以使用热插拔的组件（不关机更换磁盘，内存模块，甚至处理器）——但它必然囿于单个地理位置的桎梏。A shared-memory architecture may offer limited fault tolerance—high-end machines have hot-swappable components (you can replace disks, memory modules, and even CPUs without shutting down the machines)—but it is definitely limited to a single geographic location. 

而水平扩展 - 即分布式，即数据分布在多个节点上，有两种常见的方式：

***复制（Replication\***。数据的主从复制和备份。

***分区 (Partitioning)\***。将一个大型数据库拆分成较小的子集（称为**分区（partitions）**），从而不同的分区可以指派给不同的**节点（node）**（亦称**分片（shard）**） 

本文我们来讨论 复制。

## 5 Replication

### 5.0 Leader and Follower 

【单主从复制】

故障切换 failover 会出现很多大麻烦：\[新leader 落后于 老leader\] 

- 如果使用**异步**复制，则新主库可能没有收到老主库宕机前最后的写入操作。在选出新主库后，如果老主库重新加入集群，新主库在此期间可能会收到冲突的写入，那这些写入该如何处理？最常见的解决方案是简单丢弃老主库未复制的写入，-
- （这可能是）极其危险的操作。例如在GitHub 的一场事故中，一个过时的MySQL从库被提升为主库。数据库使用自增ID 作为主键，因为新主库的计数器落后于老主库的计数器，所以新主库重新分配了一些已经被老主库分配掉的ID作为主键。这些主键也在 Redis 中使用，主键重用使得 MySQL 和 Redis 中数据产生不一致，最后导致一些私有数据泄漏到错误的用户手中。For example, in one incident at GitHub [[13](ch05.html#Newland2012tw)], an out-of-date MySQL follower was promoted to leader. The database used an autoincrementing counter to assign primary keys to new rows,  but because the new leader’s counter lagged behind the old leader’s, it reused some primary keys that were previously assigned by the old leader. These primary keys were also used in a Redis store, so the reuse of primary keys resulted in inconsistency between MySQL and Redis, which caused some private data to be disclosed to the wrong users

#### 5.0.0 Implementation of Replication Logs

【备份 Log 的实现】

- **Statement-based replication 基于语句的复制**。记录下它执行的*每个写入请求*（**语句（statement）**
- **Write-ahead log (WAL) shipping  WAL模式** 。See Charpter 3. In either case, the log is an append-only sequence of bytes containing all writes to the database.  disanvantage: a WAL contains details of which bytes were changed in which disk blocks. This makes replication closely coupled to the storage engine. If the database changes its ***storage format*** from one version to another, it is typically not possible to run different ***versions*** of the database software on the leader and the followers. 在任何一种情况下，日志都是包含所有数据库写入的仅追加字节序列。WAL 包含哪些磁盘块中的哪些字节发生了更改。这使复制与存储引擎紧密耦合。如果数据库将其存储格式从一个版本更改为另一个版本，通常不可能在主库和从库上运行不同版本的数据库软件。
- **Logical (row-based) log replication逻辑日志复制（基于行）**。这种复制日志被称为逻辑日志，以将其与存储引擎的（物理）数据表示区分开来。MySQL的二进制日志（当配置为使用基于行的复制时）就是使用这种方法。

### 5.1 Multi-Leader Replication

【多主复制】

在每个数据中心内使用常规的主从复制；在数据中心之间，每个数据中心的主库都会将其更改复制到其他数据中心的主库中。Within each datacenter, regular leader–follower replication is used; between datacenters, each datacenter’s leader replicates its changes to the leaders in other datacenters.

例如，考虑一个由两个用户同时编辑的维基页面，如图所示。用户1将页面的标题从A更改为B，并且用户2同时将标题从A更改为C。每个用户的更改已成功应用到其本地主库。但当异步复制时，会发现冲突。单主数据库中不会出现此问题。 

![NeatReader-1688635574682.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/NeatReader-1688635574682.png)


实现冲突合并解决有多种途径，途径之一是：

-   给每个写入一个唯一的ID（例如，一个时间戳，一个长的随机数，一个UUID或者一个键和值的哈希），挑选最高ID的写入作为胜利者，并丢弃其他写入。如果使用时间戳，这种技术被称为**最后写入胜利（LWW, last write wins）**。/*还是比较危险的，git要是用这个来自动合并那就完犊子了*/

### 5.2 Leaderless Replication

#### 5.2.0 Quorums for reading and writig 法定人数

![5-10.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/5-10.png)


在上图(原文的5-10)的示例中，我们认为即使仅在 3 个副本 replicas 中的 2 个上进行处理，写入 write 仍然是成功的。如果3个副本replicas中只有一个接受了写入 write，会怎样？

如果有n个副本，每个写入必须由w节点确认才能被认为是成功的，并且我们必须至少为每个读取查询 r 个节点。 （在我们的例子中，$n = 3，w = 2，r = 2$）。只要$w + r> n$，我们期望在读取时获得最新的值，因为 r 个读取中至少有一个节点是最新的。

Limitations of Quorum Consistency 法定人数一致性的局限性

- 如果两个写入同时发生，不清楚哪一个先发生。在这种情况下，唯一安全的解决方案是合并并发写入。如果根据时间戳（最后写入胜利）挑选出一个胜者，则由于时钟偏差，写入可能会丢失。我们将在“检测并发写入”继续讨论此话题。If two writes occur concurrently, it is not clear which one happened first. In this case, the only safe solution is to merge the concurrent writes (see “Handling Write Conflicts”. If a winner is picked based on a timestamp (last write wins), writes can be lost due to clock skew . We will return to this topic in “Detecting Concurrent Writes“ /*这段是说，在时钟偏差 clock skew (指服务器之间的时间不一致)情况下, new value 可并不一定是 new value*/ 

- 如果写操作在某些副本上成功，而在其他节点上失败（例如，因为某些节点上的磁盘已满），在小于 w 个副本上写入成功。所以整体判定写入失败，但整体写入失败并没有在写入成功的副本上回滚。这意味着如果一个写入虽然报告失败，后续的读取仍然可能会读取这次失败写入的值。If a write succeeded on some replicas but failed on others (for example because the disks on some nodes are full), and overall succeeded on fewer than *w*replicas, it is not rolled back on the replicas where it succeeded. This means that if ***a write was reported as failed***, subsequent reads may or may not return the value from that write /*类似脏读，读到了该 回滚的 data*/
- 如果携带新值的节点失败。需要从旧值节点恢复。

## Conclusion

每种方法都有优点和缺点。单主复制是非常流行的，因为它很容易理解，不需要担心冲突解决。在出现故障节点，网络中断和延迟峰值的情况下，多领导者和无领导者复制可以更加稳健，但以更难以推理并仅提供**非常弱的一致性保证**为代价。

复制可以是同步的，也可以是异步的。尽管在系统运行平稳时异步复制速度很快，但是在复制滞后增加和服务器故障时要弄清楚会发生什么，这一点很重要。