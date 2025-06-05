# KAFKA LISTENERS

> `listeners`, `advertised.listeners`, 内外网分流........ 在云原生环境下，到底应该怎么配？

Spencer Ruport:

> `LISTENERS` are what interfaces Kafka binds to. `ADVERTISED_LISTENERS` are how clients can connect.

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230628230635.png)

#### Is anyone listening?

Kafka is a distributed system. Data is read from & written to the _Leader_ for a given partition, which could be on any of the brokers in a cluster. When a client (producer/consumer) starts, it will request metadata about **which broker is the leader** for a partition—and it can do this from _any_ broker. The metadata returned will include the endpoints available for the Leader broker for that partition, and the client will then use those endpoints to connect to the broker to read/write data as required.

It’s these endpoints that cause people trouble. On a _single machine, running ‘bare metal’_ (no VMs, no Docker), everything might be the hostname (or just _`localhost`_) and it’s easy. But...

The key thing is that when you run a client, **the broker you pass to it is \*just where it’s going to go and get the metadata about brokers in the cluster from\***. The actual host & IP that it will connect to for reading/writing data is based on \***the data that the broker passes back in that initial connection\***

Let's check out some config.

```
KAFKA_LISTENERS: LISTENER_BOB://kafka0:29092,LISTENER_FRED://localhost:9092
KAFKA_ADVERTISED_LISTENERS: LISTENER_BOB://kafka0:29092,LISTENER_FRED://localhost:9092
KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_BOB:PLAINTEXT,LISTENER_FRED:PLAINTEXT
KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_BOB
```

*   KAFKA\_LISTENERS

    is a comma-separated list of listeners, and the host/ip and port to which Kafka binds to on which to listen. For more complex networking this might be an IP address associated with a given network interface on a machine. The default is 0.0.0.0, which means listening on all interfaces.

    * `listeners`
*   KAFKA\_ADVERTISED\_LISTENERS

    is a comma-separated list of listeners with their the host/ip and port. This is the metadata that’s passed back to clients.

    * `advertised.listeners`
*   KAFKA\_LISTENER\_SECURITY\_PROTOCOL\_MAP

    defines key/value pairs for the security protocol to use, per listener name.

    * `listener.security.protocol.map`

_Kafka brokers communicate between themselves_, usually on the internal network (e.g. Docker network, AWS VPC, etc). Define it in (`inter.broker.listener.name`).

`kafkacat` is a useful tool for exploring this. Using `-L` you can see the metadata for the listener to which you connected.

See **`broker 0 at`**: -

*   Connecting on port 9092 (which we map as `LISTENER_FRED` LISTENER\_FRED), the broker’s address is given back as `localhost`

    ```
      $ kafkacat -b kafka0:9092 \
                 -L
      Metadata for all topics (from broker -1: kafka0:9092/bootstrap):
      1 brokers:
        broker 0 at localhost:9092
    ```
*   Connecting on port 29092 (which we map as `LISTENER_BOB` LISTENER\_BOB ), the broker’s address is given back as `kafka0`:

    ```
      $ kafkacat -b kafka0:29092 \
                 -L
      Metadata for all topics (from broker 0: kafka0:29092/0):
      1 brokers:
        broker 0 at kafka0:29092
    ```

You can also use `tcpdump` to examine the traffic from a client connecting to the broker, and spot the hostname that’s returned from the broker.

#### Why can I connect to the broker, but the client still fails?

_tl;dr_ Even if you can make the initial connection to the broker, the address returned in the metadata may still be for a hostname that is not accessible from your client.

Let’s walk this through step by step.

1. We’ve got a broker on AWS. We want to send a message to it from our laptop. We know the external hostname for the EC2 instance (`ec2-54-191-84-122.us-west-2.compute.amazonaws.com`).

We do smart things like checking that our local machine can connect to the port on the AWS instance:

```
 $ nc -vz ec2-54-191-84-122.us-west-2.compute.amazonaws.com 9092
 found 0 associations
 found 1 connections:
     1:	flags=82<CONNECTED,PREFERRED>
   outif utun5
   src 172.27.230.23 port 53352
   dst 54.191.84.122 port 9092
   rank info not available
   TCP aux info available

 Connection to ec2-54-191-84-122.us-west-2.compute.amazonaws.com port 9092 [tcp/XmlIpcRegSvc] succeeded!
```

Things are looking good! We run:

```
 echo "test"|kafka-console-producer --broker-list ec2-54-191-84-122.us-west-2.compute.amazonaws.com:9092 --topic test
```

Now…what happens next?

1. Our laptop resolves `ec2-54-191-84-122.us-west-2.compute.amazonaws.com` successfully (to the IP address 54.191.84.122), and connects to the AWS machine on port 9092
2. The broker receives the inbound connection on port 9092. _It returns the metadata to the client, with the hostname `ip-172-31-18-160.us-west-2.compute.internal`_ because this is the host name of the broker and the default value for `listeners`.
3. The client the tries to send data to the broker using the metadata. Since `ip-172-31-18-160.us-west-2.compute.internal` is not resolvable from the internet, it fails.

#### HOWTO: Connecting to Kafka on Docker

1.  Here’s the docker-compose snippet:

    ```yaml
    […]


    kafka0:
        image: "confluentinc/cp-enterprise-kafka:5.2.1"
        ports:
          - '9092:9092'
          - '29094:29094'
        depends_on:
          - zookeeper
        environment:
          […]
          # For more details see See https://rmoff.net/2018/08/02/kafka-listeners-explained/
          KAFKA_LISTENERS: LISTENER_BOB://kafka0:29092,LISTENER_FRED://kafka0:9092,LISTENER_ALICE://kafka0:29094
          KAFKA_ADVERTISED_LISTENERS: LISTENER_BOB://kafka0:29092,LISTENER_FRED://localhost:9092,LISTENER_ALICE://never-gonna-give-you-up:29094
          KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_BOB:PLAINTEXT,LISTENER_FRED:PLAINTEXT,LISTENER_ALICE:PLAINTEXT
          KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_BOB
    […]
    ```

* Clients _within_ the Docker network connect using listener “BOB”, with port 29092 and hostname `kafka0`. In doing so, they get back the hostname `kafka0` to which to connect.
* Clients _external_ to the Docker network connect using listener “FRED”, with port 9092 and hostname `localhost`. Port 9092 is exposed by the Docker container and so available to connect to. When clients connect, they are given the hostname `localhost` for the broker’s metadata, and so connect to this when reading/writing data. /_蓝绿色那块producer能按kafka0找 broker？_/
* The above configuration would _not_ handle the scenario in which a client external to Docker _and_ external to the host machine wants to connect. This is because neither `kafka0` (the internal Docker hostname) _or_ `localhost` (the loopback address for the Docker host machine) would be resolvable.

#### HOWTO: Connecting to Kafka on AWS/IaaS

The main difference is that whilst with Docker the external connections may well be just on localhost (as above), with

**Option 1 - external address IS resolvable locally**

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230629032201.png)

**Option 2 - external address is NOT resolvable locally**

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230629032226.png)

You will need to configure two listeners for Kafka:

1. Communication _within the AWS network (VPC)_. This could be inter-broker communication (i.e. between brokers),
2. External AWS traffic. This could be testing connectivity from a laptop, or simply from machines not hosted in Amazon. In both cases, the external IP of the instance needs to be used.

Here’s an example configuration:

```
listeners=INTERNAL://0.0.0.0:19092,EXTERNAL://0.0.0.0:9092
listener.security.protocol.map=INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
advertised.listeners=INTERNAL://ip-172-31-18-160.us-west-2.compute.internal:19092,EXTERNAL://ec2-54-191-84-122.us-west-2.compute.amazonaws.com:9092
inter.broker.listener.name=INTERNAL
```

#### Exploring listeners with Docker

Take a look at [https://github.com/rmoff/kafka-listeners](https://github.com/rmoff/kafka-listeners). This includes a docker-compose to bring up a Zookeeper instance along with Kafka broker configured with several listeners.

*   Listener `BOB` (port 29092) for internal traffic on the Docker network

    ```
      $ docker-compose exec kafkacat \
              kafkacat -b kafka0:29092 \
              -L
      Metadata for all topics (from broker 0: kafka0:29092/0):
      1 brokers:
        broker 0 at kafka0:29092
    ```
*   Listener `FRED` (port 9092) for traffic from the Docker-host machine (`localhost`)

    ```
      $ docker-compose exec kafkacat \
              kafkacat -b kafka0:9092 \
                      -L
      Metadata for all topics (from broker -1: kafka0:9092/bootstrap):
      1 brokers:
        broker 0 at localhost:9092
    ```
*   Listener `ALICE` (port 29094) for traffic from outside, reaching the Docker host on the DNS name `never-gonna-give-you-up`

    ```
      $ docker run -t --network kafka-listeners_default \
                  confluentinc/cp-kafkacat \
                  kafkacat -b kafka0:29094 \
                          -L
      Metadata for all topics (from broker -1: kafka0:29094/bootstrap):
      1 brokers:
        broker 0 at never-gonna-give-you-up:29094
    ```
