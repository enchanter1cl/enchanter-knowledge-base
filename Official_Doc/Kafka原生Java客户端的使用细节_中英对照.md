# Kafka原生Java客户端的使用\_中英对照

> 本文摘录和重新整理了官方文档教程中的重点，进行分析解释，并进行部分翻译

Kafka exposes all its functionality over a language independent protocol which has clients available _**in many programming languages**_. However only the Java clients are maintained as part of the main Kafka project, the others are available as independent open source projects. A list of non-Java clients is available [here](https://cwiki.apache.org/confluence/display/KAFKA/Clients). Kafka 有着各种语言的客户端，但只有 Java 客户端是官方维护的。由社区维护的其他语言客户端可以点链接查看。

> 注意：在 Kafka 官方文档中 a record (一条记录) 即指其他 MQ 语境中的 a message (一条消息)。

## 0. Producer API

[Producer Api](https://kafka.apache.org/documentation/#producerapi)

### 0.0 导包

```xml
<dependency>
	<groupId>org.apache.kafka</groupId>
	<artifactId>kafka-clients</artifactId>
	<version>3.5.0</version>
</dependency>
```

### 0.1 几个重要的类

[KafkaProducer](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/producer/KafkaProducer.html)\<K,V>

A Kafka client that publishes records to the Kafka cluster. Producer客户端。

[ProducerRecord](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/producer/ProducerRecord.html)\<K,V>

A key/value pair to be sent to Kafka. 要发送的内容，键值对。键可以不写。

[RecordMetadata](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/producer/RecordMetadata.html)

The metadata for a record that has been acknowledged by the server.

### 0.2 用例

Sending k/v pairs:

```java
// 设置属性
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("linger.ms", 1);
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer"); props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");   
// Producer
Producer<String, String> producer = new KafkaProducer<>(props);
for (int i = 0; i < 100; i++)
  producer.send(new ProducerRecord<String, String>("my-topic", Integer.toString(i), Integer.toString(i)));  
//关闭资源
producer.close();
```

The producer consists of a pool of buffer space that holds <mark style="color:purple;">records</mark> that haven't yet been transmitted to the server as well as a background I/O thread that is responsible for turning these records into requests and transmitting them to the cluster. Failure to close the producer after use will leak these resources. 生产者由一个缓冲空间池组成，该缓冲空间存放尚未传送到 server 的记录，以及一个后台 I/O 线程，负责将这些 record 变成 request ，并将其传送到 cluster。如果在使用后没有关闭生产者，将会泄露这些资源。

The [`send()`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/producer/KafkaProducer.html#send\(org.apache.kafka.clients.producer.ProducerRecord\)) method is asynchronous. When called, it adds the record to a buffer of pending record sends and immediately returns. This allows the producer to batch together individual records for efficiency. \[`send()`]方法是异步的。当被调用时，它将记录添加到一个待定记录发送的缓冲区，并**立即返回**。

The `acks` config controls the criteria under which requests are considered complete. The default setting "all" will result in blocking on the full commit of the record, the slowest setting. `acks`配置控制着请求被视为完成的标准。如果你使用默认设置 "All "，将导致记录完全提交时的阻塞，这是一个最慢的选项。

详细请参考：知识卡片 \[Kafka - Ack]\(./Knowledge Cards/Kafka - Ack.md)

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td>Kafka - Ack</td><td></td><td></td><td><a href="Knowledge Cards/Kafka - Ack.md">Kafka - Ack.md</a></td></tr></tbody></table>

`linger.ms`是说延迟多久来发送。——那设成 linger.ms=0 就是一条一发了吗？No. Under heavy load, batching will occur regardless of the linger configuration. 在重(zhong)载情况下，无论 linger.ms 配置如何，批处理都会发生。

The `key.serializer` and `value.serializer` instruct how to turn the key and value <mark style="color:orange;">**objects**</mark> the user provides with their `ProducerRecord` into <mark style="color:orange;">bytes</mark>. You can use the included [`ByteArraySerializer`](https://kafka.apache.org/35/javadoc/org/apache/kafka/common/serialization/ByteArraySerializer.html) or [`StringSerializer`](https://kafka.apache.org/35/javadoc/org/apache/kafka/common/serialization/StringSerializer.html) for simple byte or string types. 这两个属性是用来设置由 String 到 bytes 的序列化。（如果你想发送 Java 对象，可以进行其他设置。不过笔者还是建议先转成 json 字符串）

## 1. Consumer API

### 1.0 导包

```xml
<dependency>
	<groupId>org.apache.kafka</groupId>
	<artifactId>kafka-clients</artifactId>
	<version>3.5.0</version>
</dependency>
```

A client that consumes records from a Kafka cluster.

The consumer <mark style="color:yellow;">maintains TCP connections</mark> to the necessary brokers to fetch data. Failure to close the consumer after use will leak these **connections**. The consumer is not thread-safe. See [Multi-threaded Processing](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#multithreaded) for more details. 消费者维护着 TCP 长连接。用完请记得关闭资源。

### 1.1 从一个面试问题开始

在我们正式开始研究官方文档前，请先试着回答这个问题——

_**Kafka 会丢消息吗？怎样解决？**_

_**那会重复消费吗？怎样解决？**_

这个问题涉及生产端到 broker 再到消费端的一系列细节。而在本文中我们只探讨消费端。

#### 1.1.0 至少一次和最多一次

MQ 官网常见的两种语义 “at-most-onece” “at-least-once”

| code                          | semantic      |
| ----------------------------- | ------------- |
| manually commit - process msg | at-most-once  |
| auto commit                   | at-most-once  |
| process msg - manually commit | at-least-once |

| code            | semantic |
| --------------- | -------- |
| 先手动提交 - 再处理 msg | 最多消费一次   |
| 定期自动提交          | 最多消费一次   |
| 先处理 msg - 在手动提交 | 至少消费一次   |

其实特殊设置下，定期自动提交也可以 'at-least-once'，不常见，就不展开了。

### 1.2 Offsets and Consumer Position

`offset` 与消费位置

Kafka maintains a numerical offset for each record in a partition. This offset acts as a **unique identifier** of a record within that partition, and also denotes the position of the consumer in the partition. For example, a consumer which is at position 5 has consumed records with offsets 0 through 4 and will next receive the record with offset 5. Kafka 为分区中的每条记录维护一个数字偏移。这个偏移量作为该分区中一条记录的唯一标识符，同时也表示消费者在该分区中的位置。例如，一个在位置 5 的消费者已经消费了偏移量为 0 到 4 的记录，接下来将收到偏移量为 5 的记录。<mark style="color:purple;">/</mark>_<mark style="color:purple;">可以这样理解，offset 实际上就是 kafka 这个数据库中每条 record 的主键 primary key，partition 就是一张 table，不过严格讲是 分库分表 后的一张“分表”。</mark>_ <mark style="color:purple;"></mark><mark style="color:purple;">/</mark> There are actually two notions of position relevant to the user of the consumer:

The [`position`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#position\(org.apache.kafka.common.TopicPartition\)) . It will be one larger than the highest offset the consumer has seen in that partition. `position` 永远比最高的 offset 大 1.

The [`committed position`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#commitSync\(\)) is the last offset that has been stored securely. Should the process fail and restart, this is the offset that the consumer will recover to. The consumer can either automatically commit offsets **periodically**; or it can choose to control this committed position **manually** by calling one of the commit APIs (e.g. [`commitSync`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#commitSync\(\)) and [`commitAsync`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#commitAsync\(org.apache.kafka.clients.consumer.OffsetCommitCallback\))). '已提交位置'，如果 consumer 从故障恢复，会接着这个位置消费。‘提交’ 可以周期自动也可以手动。面试官超级喜欢问哦。

This distinction gives the consumer control over when <mark style="background-color:yellow;">a record is considered consumed.</mark>

### 1.3 Consumer Groups and Topic Subscriptions

消费者组与主题订阅

Kafka uses the concept of _consumer groups_ to allow a pool of processes to divide the work of consuming and processing records. These processes can either be running on the same machine or they can be distributed over many machines to provide scalability and fault tolerance for processing. All consumer instances sharing the same `group.id` will be part of the same consumer group. 消费者组，就是一池进程一起分担消息消费。

Each consumer in a group can dynamically set the list of topics it wants to subscribe to through one of the [`subscribe`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#subscribe\(java.util.Collection,org.apache.kafka.clients.consumer.ConsumerRebalanceListener\)) APIs. .....each partition is assigned to exactly one consumer in the group. So if there is a topic with four partitions, and a consumer group with two processes, each process would consume from two partitions. partition 会被多对一地分配给组内 consumer。 /_单纯为了分工，想象一下你有个几千万行的大文件，让 thread pool 中 thread-0 处理process 0-999行， thread-1 处理 1000-1999行......._/

\[ “臭名昭著”的 rebalancing ]

（为什么说他“臭名昭著”呢，因为 bug 很多，而且是消息顺序消费的大杀手。生产环境下一般尽量防止 Kafka 自动 rebalancing）

If a process fails... Similarly, if a new consumer process joins the group.... This is known as _rebalancing_ the group and .... Group rebalancing is also used when **new partitions** are added or when a new topic matching a [`subscribed regex`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#subscribe\(java.util.regex.Pattern,org.apache.kafka.clients.consumer.ConsumerRebalanceListener\)) is created. The group will automatically detect the new partitions through periodic _**metadata refreshes**_ and assign them to members of the group. 当一个 consumer 进程崩溃时，当一个新 consumer 进程加入 group 时，当新 partition 加入 topic 时......

Conceptually you can think of a consumer group as being a single logical subscriber that happens to be made up of multiple processes. 从概念上讲，你可以把消费者组看作是一个由多个进程组成的单一逻辑订阅者。 Kafka naturally supports having any number of groups for **a** given topic. group 对 topic 可以多对一。

如果你想获得类似于传统消息系统中 pub-sub 的语义semantics，你可以设一个 消费者进程process 一个 group，这样每个 process 都会订阅到 topic 的所有记录。

### 1.4 Detecting Consumer Failures

监测宕机的消费者

After subscribing to a set of topics, the consumer will automatically join the group when [`poll(Duration)`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#poll\(java.time.Duration\)) is invoked. The poll API is designed to ensure consumer liveness. As long as you continue to call poll, the consumer will stay in the group and continue to receive messages from the partitions it was assigned. Underneath the covers, the consumer sends periodic heartbeats to the server. If the consumer crashes or is unable to send heartbeats for a duration of `session.timeout.ms`, then the consumer will be considered dead and its partitions will be reassigned. 如果超过 `session.timeout.ms` 设置的时延消费者还没发送心跳，Kafka 就认为它死了。

It is also possible that the consumer could encounter a "livelock" situation where it is continuing to send heartbeats, but no progress is being made. To prevent the consumer from holding onto its partitions indefinitely in this case, we provide a liveness detection mechanism using the `max.poll.interval.ms` setting. Basically if you don't call poll at least as frequently as the configured max interval, then the client will proactively leave the group so that another consumer can take over its partitions. When this happens, you may see an offset commit failure (as indicated by a [`CommitFailedException`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/CommitFailedException.html) thrown from a call to [`commitSync()`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#commitSync\(\))). This is a safety mechanism which guarantees that only active members of the group are able to commit offsets. So to stay in the group, you must continue to call poll. 消费者也有可能遇到 "livelock "的情况，即它继续发送心跳，但没有任何进展。为了防止消费者在这种情况下无限期地坚持其分区 partition，我们使用 `max.poll.interval.ms` 机制。基本上，如果你调用 poll() 的频率跟不上，那么你这个 client 将主动离开 group，以便另一个消费者可以接管你的 partition 。此时，你可能会看到偏移 offset 提交失败（如\[CommitFailedException\`]所示）/_毕竟这个 consumer 已经被踢出 group 了_/。这是一个安全机制，它保证只有 group 内的活跃成员 active member 才能提交偏移量。所以要想留在 group 内，必须继续调用 poll()。

Settings to control the behavior of the poll loop 两个可以控制轮询的选项:

1. `max.poll.interval.ms`: By increasing the interval between expected polls, you can give the consumer more time to handle a batch of records returned from [`poll(Duration)`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#poll\(java.time.Duration\)). 轮询间隔。
2. `max.poll.records`: To limit the total records returned from a single call to poll. 限制单次调用轮询所返回的总记录数。

\[ Kafka 丢消息的时刻 ]

For use cases where message processing time varies unpredictably, neither of these options may be sufficient. 对于消息处理时间变化不可预测的用例，这两个选项可能都不够。 consumer maybe continue calling [`poll`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#poll\(java.time.Duration\)) while the processor is still working. (没处理完就 call poll()). Some care must be taken to ensure that committed offsets do not get ahead of the actual position. /_commiteed offsets 提前于实际位置，即丢消息_/ Typically, you must disable <mark style="background-color:red;">automatic commits</mark> and <mark style="background-color:red;">manually commit</mark> processed offsets for records only after the thread has finished handling them (depending on the delivery semantics you need). 你必须禁用自动提交，并在线程处理完记录后才手动提交处理过的偏移量（取决于你需要的交付语义） Note also that you will need to [`pause`](https://kafka.apache.org/35/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#pause\(java.util.Collection\)) the partition so that no new records are received from poll until after thread has finished handling those previously returned. 还要注意的是，你需要`pause`分区，以便在线程处理完之前返回的记录之前，不会从轮询中收到新的记录。

### 1.5 用例

#### 1.5.0 Automatic Offset Committing

自动提交 offset

```java
     // 设置属性
     Properties props = new Properties();
     props.setProperty("bootstrap.servers", "localhost:9092");
     props.setProperty("group.id", "test");
     props.setProperty("enable.auto.commit", "true");
     props.setProperty("auto.commit.interval.ms", "1000");
     props.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     props.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     // Consumer
     KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
     // 订阅两个主题
     consumer.subscribe(Arrays.asList("foo", "bar"));
     while (true) {
         ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
         for (ConsumerRecord<String, String> record : records)
             System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
     }
 
```

The connection to the cluster is bootstrapped by specifying a list of one or more brokers to contact using the configuration `bootstrap.servers`. 该属性用来指定 broker 集群地址。

The deserializer settings specify how to turn <mark style="color:orange;">bytes</mark> into <mark style="color:orange;">objects.</mark> For example, by specifying string deserializers, we are saying that our record's key and value will just be simple strings. bytes 反序列化为 objects 对象, 本例中是反序列化为 String 对象.

#### 1.5.1 Manual Offset Control

手动提交 offset

```java
     props.setProperty("enable.auto.commit", "false");
     final int minBatchSize = 200;
     List<ConsumerRecord<String, String>> buffer = new ArrayList<>();
     while (true) {
         ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
         for (ConsumerRecord<String, String> record : records) {
             buffer.add(record);
         }
         if (buffer.size() >= minBatchSize) {
             insertIntoDb(buffer);
             consumer.commitSync();
             buffer.clear();
         }
     }
```

In this example, When we have enough records batched, we will insert them into a database. If we allowed offsets to auto commit as in the previous example, records would be considered consumed after they were returned to the user in `poll`. 在本例中，每攒一批 records 就进行一次处理——插一次库。 It would then be possible for our process to fail after batching the records, <mark style="color:orange;">but before they had been inserted into the database (上一例的语义：轮询来了就是消费了。poll 来了就是 consumed 了。)(举个不太恰当的例子哈.....健身教程视频最常见的弹幕就是......“收藏了就是练了”)。</mark>

To avoid this, we will manually commit the offsets only after the records have have been inserted into the database. This raises the opposite possibility: the process could fail in the interval after the insert into the database but before the commit (even though this would likely just be a few milliseconds, it is a possibility). The process that took over consumption would 会重复消费，重复插库。 Used in this way Kafka provides what is often called <mark style="background-color:red;">"at-least-once"</mark> delivery guarantees.

你甚至还可以自主控制提交的 offset 是多少：

```java
long lastOffset = partitionRecords.get(partitionRecords.size() - 1).offset();
consumer.commitSync(Collections.singletonMap(partition, new OffsetAndMetadata(lastOffset + 1)));
```

#### 1.5.2 Manual Partition Assignment

手动分配分区

```java
     String topic = "foo";
     TopicPartition partition0 = new TopicPartition(topic, 0);
     TopicPartition partition1 = new TopicPartition(topic, 1);
     consumer.assign(Arrays.asList(partition0, partition1));
```

Consumer failures will not cause assigned partitions to be **rebalanced**.

这样消费者崩溃也不会被 rebalance.

## 2. Summary

生产者端，我们用 `KafkaProducer<K, V>` 来发送消息。

消费者端，我们用 `KafkaConsumer<K, V>` 来建立长连接监听特定 topic 下特定或非特定 partition 的消息。请注意 `offset` 提交时机的控制，来解决丢消息与重复消费问题。
