
业界流行的分布式锁实现，一般有这 3 种方式：
-   基于关系型数据库实现的分布式锁。
-   基于 Redis 实现的分布式锁。
-   基于 Zookeeper 实现的分布式锁。

# 0. RMDB 数据库实现分布式锁

0 基于表 （锁表，很少使用）（我 github 的 taxi 仓库演示了这种，利用了主键冲突） 
1 基于悲观锁。要使用悲观锁，我们必须关闭 Mysql 数据库的自动提交属性，因为 MySQL 默认使用 autocommit 模式，也就是说，当你执行一个更新操作后，MySQL 会立刻将结果进行提交。set autocommit=0;
2 基于乐观锁。顾名思义，就是很乐观，每次更新操作，都觉得不会存在并发冲突，只有更新失败后，才重试。

## 0.0 **悲观锁实现思路**？

需要利用到数据库的事务机制。

1. 在对任意记录进行修改前，先尝试为该记录加上排他锁（exclusive locking）。
   
2. 如果加锁失败，说明该记录正在被修改，那么当前查询可能要等待或者抛出异常。 具体响应方式由开发者根据实际需要决定。 (/_也就是下面第1步 for update 的时候 会发现加锁失败_/)
   
3. 如果成功加锁，那么就可以对记录做修改，事务完成后就会解锁了。
   
4. 其间如果有其他对该记录做修改或加排他锁的操作，都会等待我们解锁或直接抛出异常。

假设下面多条线程在竞争同一商品：

```sql
# 0.开始transaction
 BEGIN;/BEGIN WORK;/START TRANSACTION;

# 1.查询出商品信息
 SELECT status FROM t_goods WHERE id=1 FOR UPDATE;

# 2.根据商品信息生成订单
 INSERT INTO t_order(id, goods_id) VALUES (NULL, 1);

# 3. 修改商品status为1
 UPDATE t_goods SET status=1;

# 4.提交事务
 COMMIT;/COMMIT WORK;
```

上面的查询语句中，我们使用了 **SELECT…FOR UPDATE** 的方式，这样就通过开启排他锁 exclusive lock 的方式实现了悲观锁。此时在 t_goods 表中，id为 1 的 那 row 数据就被我们锁定了，其它的事务 tx 必须等本次事务 tx 提交之后才能执行。这样我们可以保证当前的数据不会被其它事务修改。

上面我们提到，使用 SELECT ... FOR UPDATE 会把数据给锁住，不过我们需要注意一些锁的级别，MySQL InnoDB 下，行级锁都是基于索引 index 的，如果一条 SQL 语句用不到 index 是不会使用行级锁 row lock 的，会使用表级锁 table lock 把整张表锁住，这点需要注意。

## 0.1 乐观锁实现思路?

```sql
# 1.查询出商品信息
 SELECT (status, version) FROM t_goods WHERE id=1;

# 2.根据商品信息生成订单
 INSERT INTO t_order(id, goods_id) VALUES (NULL, 1);

# 3.改商品status为0
 UPDATE t_goods SET status=0, version=version+1  WHERE id=1 AND version=#{version};
```

# 1. Redis 实现分布式锁

redis distributed lock

## 1.0 SET NX PX

`SET orderId:lock 0xx9p NX PX 30000`  （30s 后 expire）

释放锁 release lock 时要特别注意！ 在删除 key 之前，一定要判断服务 A 持有的 value 与 Redis 内存储的 value 是否一致。如果贸然使用服务 A 持有的 key 来删除锁，可能会误将服务 B 的锁释放掉。

```lua
 if redis.call("get", KEYS[1]==ARGV[1]) then  
     return redis.call("del", KEYS[1])  
 else  
     return 0  
 end
```

## 1.1 基于 RedLock

假设有两个服务 A、B 都希望获得锁，有一个包含了 5 个 redis master 的 Redis Cluster，执行过程大致如下:

1. 客户端获取当前时间戳，单位: 毫秒
   
2. 服务 A 轮询个 master 节点，尝试创建锁。(这里锁的过期时间比较短，一般就几十毫秒) RedLock 算法会尝试在大多数节点上分别创建锁，大多数节点指过半.
   
3. 客户端计算成功建立完锁的时间，如果建锁时间小于超时时间，就可以判定锁创建成功。如果锁创建失败，则依次(遍历 master 节点)删除锁。
   
4. 只要有其它服务创建过分布式锁，那么当前服务就必须轮询尝试获取锁。

（用锁的几个原则：互斥性，防死锁，自己解自己的锁，容错性如 n 个redis）

![RedLock1.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/RedLock1.png)

注意，如果某一台 redis 挂了，要延迟重启。比如下图，线程一在 1 2 3 号 redis 上半数同意抢到锁后，3 号还没来得及持久化就挂了，那么.......

![RedLock立即重启.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/RedLock%E7%AB%8B%E5%8D%B3%E9%87%8D%E5%90%AF.png)

## 1.2 基于 Redis 的客户端如 Redisson

> 这里Redis的客户端（Jedis, Redisson, Lettuce等）**都是基于上述两类形式来实现分布式锁**的，只是两类形式的封装以及一些优化（比如 Redisson 的 watch dog)。

以基于 Redisson 实现分布式锁为例（支持了 单实例、redis 哨兵、redis cluster、redis master-slave 等各种部署架构）：

**特色**？

1. redisson 所有指令都通过 lua 脚本执行，保证了操作的原子性
   
2. redisson 设置了watchdog 看门狗，**“看门狗”的逻辑保证了没有死锁发生**
   
3. redisson 支持 Redlock 的实现方式。
   

**过程**？

1. 线程去获取锁，获取成功: 执行 lua 脚本，保存数据到 redis 数据库。
   
2. 线程去获取锁，获取失败: 订阅了解锁消息，然后 retry 去获取锁。
   

**watch dog 自动延时机制**？

client A 加锁的锁 key 默认生存时间只有 30 秒，如果超过了 30 秒，client A 还想一直持有这把锁，怎么办？其实只要 client A 一旦加锁成功，就会启动一个 watch dog 看门狗，它是一个后台线程，会每隔 10 秒检查一下，如果 client A 还持有锁 key，那么就会不断的延长锁 key 的生存时间。

**可重入**？

每次 lock 会调用 incrby，每次 unlock 会减一。

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230708120117.png)


## 1.3 进一步理解

借助 Redis 实现分布式锁时，有一个共同的缺陷：当获取锁被拒绝后，需要不断的循环，**重新发送获取锁(创建key)的请求**，直到请求成功。这就造成空转，浪费宝贵的 CPU 资源。

参考 [锁 0.1 章节](./并发编程/锁)

## 1.4 到底用几台 Redis？

我怕单台 redis 挂掉，所以单主从复制 single leader replication 地加 redis 可以吗？
不可以。 官网不推荐。因为 redis 单主从复制 single leader replication 是异步 async 的，不是**强一致** linearizability 的.

> Coordination services like Apache ZooKeeper and etcd are often used to implement **distributed locks** and leader election. They use consensus algorithms to implement linearizable operations.

顺便复习下 Redis 基础知识

什么是哨兵？
哨兵是观察者 Observer，通知 failover 后领导者选举 leader election 的。  

那用 redis cluster 可以做分布式锁的吗？
不可以。 redis cluster 是用来做 partition 的不是 replication 的。比如 5G 数据要往 redis 放，一个放 3G 一个放2G。 

综上所述，用 redis 做 distributed lock， 最好就用一个单示例 instance.。注意，不能开自动重启，会出现重复锁。

关于 Redis HA 集群我会找时间另开新文讲解。