# Kafka - Ack

\[acks 确认机制]

ack 直接关系 Kafka 集群的**吞吐量**和**可靠性**。而吞吐量和可靠性，就像硬币的两面，两者不可兼得，只能平衡。

acks 参数制定了**必须有多少个 partition 收到 message, producer 才认为该 message 是写入成功的**。

(笔者在这里强烈推荐阅读 《Design Data Intensive Application》（简称DDIA）的第二部分第五章节 - 5 Leaderless replication - 5.4.1 Quorums for reading and writing)

Quorums. Instead of majority vote, Kafka dynamically maintains a set of in-sync replicas (ISR名单). A write to a Kafka partition is not considered committed until _all_ in-sync replicas have received the write.

* `acks=0` . 完全不期待回应。**发送它，然后忘记它**。我发了就当成功了。所以可以以网络支持的最大速度发送消息，从而达到很高的吞吐量 throughput。
* `acks=1`. 只要 cluster 的 leader 接收到了 message，就会向 producer 发送一个成功响应的 ack，此时 producer 接收到 ack 之后就可以认为该 message 是写入成功的。
* `acks=all(-1)`, 表示只有所有参与复制的节点 nodes 全部收到 message 时，producer 才会接收到来自 kafka 的响应.
