# 内外网访问 Kafka Listeners

> `listeners`, `advertised.listeners`, 内外网分流........ 在云原生环境下，到底应该怎么配？通过动手实操了解

Spencer Ruport:

> `LISTENERS` are what interfaces Kafka binds to. `ADVERTISED_LISTENERS` are how clients can connect.

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230628230635.png)

![kafka-listener.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/kafka-listener.png)&#x20;

注意图和下文颜色是对应的。

我们会用到一个很好用的小工具 \[kcat]\([edenhill/kcat: Generic command line non-JVM Apache Kafka producer and consumer (github.com)](https://github.com/edenhill/kcat))。

跑一个 container, ENV 配置如下

| key                                           | value                                                       |
| --------------------------------------------- | ----------------------------------------------------------- |
| ALLOW\_PLAINTEXT\_LISTENER                    | yes                                                         |
| KAFKA\_CFG\_LISTENERS                         | PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094       |
| KAFKA\_CFG\_ADVERTISED\_LISTENERS             | PLAINTEXT://kafka-bro-cn:9092,EXTERNAL://localhost:9094     |
| KAFKA\_CFG\_LISTENER\_SECURITY\_PROTOCOL\_MAP | CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT |

container name 改名为 kafka-bro-cn (container name 的意思), 网络改名为 app-tier

## 1. 从宿主机访问

即 <mark style="background-color:green;">在 host 上用 kcat</mark>

kcat 联系的是 `listeners` 中的一号监听器 EXTERNAL

```shell
kcat -b 127.0.0.1 -L
[root@ecs-6fd7 ~]# kcat -b 127.0.0.1:9094 -L
Metadata for all topics (from broker -1: 127.0.0.1:9094/bootstrap):
 1 brokers:
  broker 1 at localhost:9094 (controller)
 2 topics:
  topic "test" with 1 partitions:
    partition 0, leader 1, replicas: 1, isrs: 1
  topic "__consumer_offsets" with 50 partitions:
    partition 0, leader 1, replicas: 1, isrs: 1
    partition 1, leader 1, replicas: 1, isrs: 1
    partition 2, leader 1, replicas: 1, isrs: 1
    ......
```

注意看 broker  1 at  localhost:9094  是 `advertise.listener` list 中的 Listener EXTERNAL 的内容。

## 2. 从同网络的其他容器访问

在 <mark style="background-color:orange;">other containers on the same Docker network 用 kcat,</mark>

kcat 联系的是 `listeners` 中二号监听器 PLAINTEXT

```shell
docker run -it \
 --network=app-tier \
 edenhill/kcat:1.7.1 \
  -b kafka-bro-cn:9092 -L
```

结果如下

```shell
Metadata for all topics (from broker 1: kafka-bro-cn:9092/1):
 1 brokers:
  broker 1 at kafka-bro-cn:9092 (controller)
 2 topics:
  topic "test" with 1 partitions:
    partition 0, leader 1, replicas: 1, isrs: 1
    ......

```

注意看第2, 3行 1 brokers: broker 1 at kafka-bro-cn:9092 是 `advertise.listener` list 中的 Listener PLAINTEXT  的内容

## 3. 从同子网的其他宿主机访问

3 在 other host (同一子网subnet 中)  上用 kcat 怎么办呢？

访问该 host 内网ip

```shell
[root@ecs-6fd7-fb07 kcat]# kcat -b 172.31.0.16:9094 -L
Metadata for all topics (from broker -1: 172.31.0.16:9094/bootstrap):
 1 brokers:
  broker 1 at localhost:9094 (controller)
 0 topics:

```

(我重启了container 所以 topics 变 0了)

他给的 `advertise.listener` 是 list 中(确切说是 map )的 二号 listener EXTERNAL, 内容是 localhost:9092

所以 第一道关过了 第二道可过不了

用 console-producer

```shell
[root@ecs-6fd7-fb07 kafka]bin/kafka-console-producer.sh --bootstrap-server 172.31.0.16:9094 --topic test1
>fist
[2023-06-29 18:13:19,104] WARN [Producer clientId=console-producer] Connection to node 1 (localhost/127.0.0.1:9094) could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)
[2023-06-29 18:13:19,204] WARN [Producer clientId=console-producer] Connection to node 1 (localhost/127.0.0.1:9094) could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)
......
```

改 broker 的 ENV ---- EXTERNAL://172.31.0.16:9094  好了

第一道：

```shell
[root@ecs-6fd7-fb07 bin]# kcat -b 172.31.0.16:9094 -L
Metadata for all topics (from broker 1: 172.31.0.16:9094/1):
 1 brokers:
  broker 1 at 172.31.0.16:9094 (controller)
 0 topics:
```

第二道：

```shell
[root@ecs-6fd7-fb07 bin]# ./kafka-console-producer.sh --bootstrap-server 172.31.0.16:9094 --topic test1
>this is first event
[2023-06-29 18:18:49,789] WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 4 : {test1=UNKNOWN_TOPIC_OR_PARTITION} (org.apache.kafka.clients.NetworkClient)
>this is second event
>this is thr^Hird event
>

```

虽然报了个 WARN 但其实写进去了:

```shell
[root@ecs-6fd7-fb07 kafka]# bin/kafka-console-consumer.sh --topic test1 --from-beginning --bootstrap-server 172.31.0.16:9094
this is first event
this is second event
this is third event

```

那么如果我想从外网访问呢？比如我的电脑 laptop

## 4. 公网访问

other host (从公网)

在 KAFKA\_CFG\_LISTENERS 添加一项 LAPTOP://:9095

在 KAFKA\_CFG\_ADVERTISED\_LISTENERS 添加 LAPTOP://公网IP:9095

把 9095 映射出去 `port: -'9095:9095'` （想象一下路由器的路由表）

```shell
kafka-console-consumer.bat --topic test1 --from-beginning --bootstrap-server 公网IP:9095
```
