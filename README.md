# Apache Kafka
- This project contains an example of a docker compose yml for an Apache Kafka server, it uses [Bitnami Image](https://hub.docker.com/r/bitnami/kafka)

### What is Apache Kafka?
- Apache Kafka is an open-source distributed event store and stream-processing platform that was originally developed by LinkedIn and later became part of the Apache Software Foundation.

### What are the essential main components involved in Kafka communication?
-   There are 3 main components
    1. Producer: Sends to the broker. It is the source of the messages.
    2. Broker: It is a message store and represents the point of contact between producer and consumer.
    3. Consumer: Pulls from Broker. (This differs from other messaging systems like RabbitMQ or ActiveMQ where the broker pushes the message to the consumers)

### What is a Topic?
- A topic is a destination on the broker where producers send the messages to. It is a collection of related events that get persisted on disk. These events can be stored in the topic permanently or for a configurable amount of time. The default retention period is 1 week. Topics in Kafka are the high-level structure. Topics are divided into one or more partitions, which allows the data to be scaled across a Kafka cluster for parallel processing.

### What is a Log?
- A log, in the context of Kafka, refers to the low-level representation of a partition within a topic. A Log is an append-only structure. This means that new events are added to the end of the Log. The log is also immutable which means that once those events are added the content cannot be changed. The log also contains an offset.

### What is a the offset of a Log?
- Kafka maintains a numerical offset for each record in a partition. This offset acts as a unique identifier of a record within that partition, and also denotes the position of the consumer in the partition.

### What is a message or a record in Kafka?
- A message is something sent over asynchronous communication channel which can contain a request or an event. A message is compound of:
    1. Key: Allows for ordering of events. It is optional, defaults to null. Messages with the same key are guaranteed to be placed in the same partition.
    2. Headers: Holds metadata information. It consists of ordered key-value pairs. The key is a string identifier. The value is any serialized object.
    3. Value: It represents a payload and can be a request or an event. It can be in a number of formats like string, json or Avro. Default max size is 1 MB.
    4. Timestamp: Creation time or ingestion time.

### What is a Consumer Group?
- Consumer group is an identifier or label that is applied to a consumer. Each consumer in a consumer group independently maintains its position (offset) in each partition. The assignment of partitions to consumers within a group ensures that each message is processed once and only once by the group, as each partition is only consumed by one consumer in the group.

### What is a segment on apache kafka?
- A segment in Apache Kafka refers to a part of a Kafka topic's partition. Kafka topics are divided into partitions for scalability and parallel processing, and each partition is further divided into segments. These segments are essentially the files where Kafka stores its data. Here's a more detailed breakdown:

    1. Partition Structure: A Kafka topic can have multiple partitions, which allow the topic to handle more data by splitting it across multiple servers. Each partition is an ordered, immutable sequence of records that is continually appended to—a commit log. Partitions enable topics to scale horizontally across the Kafka cluster.

    2. Segments and Files: Within a partition, data is further subdivided into segments. Each segment consists of two main files:

        - Log File: This file stores the actual message data (records) written to Kafka. The records within a segment file are appended in an ordered fashion, where each new record is added to the end of the log file.

        - Index File: Accompanying each log file, there's an index file that stores metadata about the records in the log file, such as offsets (a unique identifier for each record within a partition) and positions within the log file. This index allows Kafka to quickly locate and retrieve records.

     3. Segment Size and Retention: Segments have a maximum size, and when the size is reached, Kafka will seal the current segment and create a new one for future records. This segmentation allows Kafka to manage data more efficiently, including cleanup policies for data retention. Kafka supports different policies for data retention, such as retaining data for a specified time period or until the log reaches a certain size, after which old segments can be deleted or compacted.

     4. Compaction: Kafka also supports log compaction for topics, which ensures that the log contains only the latest value for each key within a partition. During compaction, old segments are rewritten, removing records with keys that have newer values in subsequent segments. This is particularly useful for maintaining the latest state of data without growing the log indefinitely.

### Why does Kafka use a pull-based message consumer?
- Unlike traditional messaging systems that often push messages to consumers, Kafka employs a pull model where the consumers request (or pull) messages from the broker when they are ready to process them. 

- The main advantages of this approach are:

    1. Consumer Pacing: Consumers can fetch messages at their own pace, allowing for efficient processing without being overwhelmed by incoming messages.

    2. Batching: Consumers can pull a batch of records from Kafka in a single request, which is more efficient than fetching records one at a time. This is particularly useful for high-throughput consumers or when processing latency is a critical factor.

    3. Scalability and Fault Tolerance: The pull model contributes to Kafka's scalability and fault tolerance. Consumers can be added or removed without impacting the brokers or other consumers. If a consumer fails, its workload can be redistributed to other consumers in the group without losing data, thanks to the consumer's control over message fetching and offset management.

### How Kafka replication works?
- Apache Kafka's replication mechanism is designed to ensure that messages are safely stored across multiple brokers in a Kafka cluster, providing fault tolerance and high availability. Here's a simplified explanation of how replication works in Kafka:
    1. Topics and Partitions:
    Each Kafka topic is divided into one or more partitions. This division allows for the parallel processing of data. Each partition can be replicated across multiple brokers to ensure data redundancy and availability.

    2. Replication Factor:
    The replication factor specifies the number of copies (replicas) that exist for each partition. For example, a replication factor of 3 means there are three copies of each partition. These replicas are distributed across different brokers in the cluster.

    3. Leader and Follower Replicas:
    For each partition, one of the replicas is designated as the leader, and the rest as followers. All produce (write) and consume (read) operations for a partition are handled by the leader replica. The followers replicate the leader's log. They pull messages from the leader and store them locally. Followers do not serve client requests directly.

    4. Producing Messages:
    Producers send messages to the broker that is the leader for the partition to which the message is being written. The leader appends the message to its log.

    5. Replication of Messages:
    After the leader has appended the message to its log, the follower replicas pull this message and append it to their own logs. Replication to followers is asynchronous by default, but Kafka also supports synchronous replication to ensure that a message is replicated to a certain number of followers before acknowledging the message as committed to the producer.

    6. High Watermark:
    Kafka maintains a high watermark for each partition, which is the offset of the last message that was successfully replicated to all in-sync replicas (ISRs). Consumers can only read messages up to the high watermark to ensure that they only consume messages that have been replicated and are unlikely to be lost if a broker fails.

    7. In-Sync Replicas (ISRs):
    A subset of the replicas are designated as "in-sync" based on their ability to keep up with the leader. Only those followers that have fully replicated the leader's log up to a certain point are considered in-sync. If a leader replica fails, Kafka will elect a new leader from the set of in-sync replicas.

    8. Leader Election:
    If the current leader of a partition becomes unavailable, Kafka will automatically elect a new leader from among the in-sync follower replicas. The election ensures minimal disruption and maintains data availability and consistency.

### What is the default partitioning strategy?
- With key --> hash(key) % number_of_partitions
- No key: 
  - Kafka version < 2.4 --> round-robin
  - Kafka version >= 2.4 --> sticky partition based on the capacity in the batch.

### What are the producer guarantees settings?
- Apache Kafka provides several configuration settings for producers to control the durability, reliability, and consistency of message delivery. These settings allow you to balance between performance and the guarantees you need for your application. Here are the key producer guarantee settings in Kafka:
    - acks (Acknowledgment levels):
        - acks=0 (None) The producer will not wait for any acknowledgment from the server before considering the request complete. This setting provides the lowest latency but the weakest durability guarantees because messages can be lost if the server fails before the messages are written to the disk.
        - acks=1 (Leader) The producer will wait for the leader replica to acknowledge the message. This ensures that the message is not lost unless the leader crashes immediately after acknowledging the message but before replicas have copied it. This provides a moderate balance between latency and durability.
        - acks=-1 (All) The producer will wait for the full set of in-sync replicas to acknowledge the message. This provides the strongest durability guarantees at the cost of higher latency.
    - retries: This setting specifies the number of retries the producer will make if a temporary failure occurs while sending a message. The default value is 2147483647 (effectively infinite retries), which, combined with the proper retry backoff, can help ensure messages are eventually delivered in the face of transient failures.
    - retry.backoff.ms: This configures the delay between retries. It allows the producer to wait for a specified amount of time before retrying a failed send operation, which can be useful to give transient failures (like a temporary network issue) time to resolve.
    - max.in.flight.requests.per.connection: This setting limits the number of unacknowledged requests the client will send on a single connection before blocking. Setting it to 1 ensures that messages are sent sequentially, which can be important for maintaining order, but it may also reduce throughput.
    - enable.idempotence: When set to true, this ensures that messages are delivered exactly once to a particular partition during the lifetime of a single producer session. This is achieved by preventing the producer from sending duplicate messages as a result of retries. It requires acks to be set to all and max.in.flight.requests.per.connection to be set to 5 or lower.

### Kafka configuration good practices
- auto.create.topics.enable: false (default true) In production environments, it's often recommended to set auto.create.topics.enable to false. This is because automatic topic creation can lead to operational issues, such as unintended topic creation due to typos in topic names, which can complicate cluster management. It also means that topics will be created with default configurations, which may not be optimal for all use cases.

### How do consumers know from what point to consume messages from a topic partition?
- The latest consumed offset for each topic partition is tracked for each consumer group

### If there are more consumer instances in a consumer group than partitions in a given partition, then how many consumers instances will be assigned the same partition?
- Only one consumer can be assigned to a topic partition at any one time. If there are more consumers than partitions, than those consumers that have no partitions assigned are idle.

### What is consumer group rebalance?
- It is a process that redistributes the partitions of a topic among the consumers in a consumer group to ensure an even load distribution.

### When consumer group rebalance is triggered?
- It is triggered when:
  1. It detect a failed consumer.
  2. A new consumer joins the consumer group
  3. Partitions Added to a Topic
  4. Manual Trigger (An administrator can also manually trigger a rebalance through various administrative actions or configurations changes that affect the group.)

### What mechanisms are used by the broker to detect a failed consumer?
- Heartbeating
- Polling time out

### How does Kafka guarantee message ordering?
- It guarantees message ordering by ensuring all messages with the same key are witten to the same partition. As a topic partition will only ever have a single consumer, the messages on the partition are guaranteed to be consumed in order.

### Start Kafka server
```bash
docker-compose-up
```

### Create a consumer that listens to a topic
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my.first.topic
```

### Create a producer that sends messages to a topic
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my.first.topic
```

### Create a producer that sends messages to a topic with a key separated by ':' example Key:Payload
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my.first.topic --property parse.key=true --property key.separator=:
```
### Create a consumer that listens to a topic and print key
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my.first.topic --property print.key=true --property key.separator=:
```

### List created topics
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-topics.sh --bootstrap-server localhost:9092 --list
```

### Create a new topic
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-topics.sh --bootstrap-server localhost:9092 --create --topic my.new.topic
```

### Show information about a topic
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic my.new.topic
```

### Alter a topic so that it has 3 partitions
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic my.new.topic --partitions 3
```

### Delete a topic
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic my.new.topic
```

### Create a topic with replication factor 3 and 2 partitions
```bash
docker run -it --rm bitnami/kafka:3.6.1 kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 2 --topic test-topic
```

### Create a topic with 5 partitions
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-topics.sh --bootstrap-server localhost:9092 --create --topic demo.topic --partitions 5
```

### Verify that it has 5 partitions
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic demo.topic
```

### List Consumer Groups
- Consumer groups aren't explicity created, instead they get created when a consumer connects to a topic or partition, when we start a consumer we can specify the name of the consumer group
- If no consumer group is specified the system will generate one because all consumers must belong to a consumer group
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
```

### Create a consumer specifying the consumer Groups
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic demo.topic --group my.new.group
```

### Show information about the consumer group
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my.new.group
```

### Show the state of the consumer group
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my.new.group --state
```

### Show the members of a consumer group
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my.new.group --members
```
