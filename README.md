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
- A topic is a destination on the broker where producers send the messages to. Messages can be stored permanently or for a configurable amount of time. Topics in Kafka are the high-level structure. Topics are divided into one or more partitions, which allows the data to be scaled across a Kafka cluster for parallel processing.

### What is a Log?
- A log, in the context of Kafka, refers to the low-level representation of a partition within a topic. A Log is an append-only structure. This means that new events are added to the end of the Log. The log is also immutable which means that once those events are added the content cannot be changed. The log also contains an offset.

### What is a the offset of a Log?
- Kafka maintains a numerical offset for each record in a partition. This offset acts as a unique identifier of a record within that partition, and also denotes the position of the consumer in the partition.

### What is a message or a record in Kafka?
- A message is something sent over asynchronouse communication channel which can contain a request or an event. A message is compound of:
    1. Key: Allows for ordering of events. It is optional, defaults to null. Messages with the same key are guaranteed to be placed in the same partition.
    2. Headers: Holds metadata information. It consists of ordered key-value pairs. The key is a string identifier. The value is any serialized object.
    3. Value: It represents a payload and can be a request or an event. It can be in a number of formats like string, json or Avro. Default max size is 1 MB.

### What is a Consumer Group?
- Consumer group is an identifier or label that is applied to a consumer. Each consumer in a consumer group independently maintains its position (offset) in each partition. The assignment of partitions to consumers within a group ensures that each message is processed once and only once by the group, as each partition is only consumed by one consumer in the group.

### Why does Kafka use a pull-based message consumer?
- Unlike traditional messaging systems that often push messages to consumers, Kafka employs a pull model where the consumers request (or pull) messages from the broker when they are ready to process them. 

- The main advantages of this approach are:

    1. Consumer Pacing: Consumers can fetch messages at their own pace, allowing for efficient processing without being overwhelmed by incoming messages.

    2. Batching: Consumers can pull a batch of records from Kafka in a single request, which is more efficient than fetching records one at a time. This is particularly useful for high-throughput consumers or when processing latency is a critical factor.

    3. Scalability and Fault Tolerance: The pull model contributes to Kafka's scalability and fault tolerance. Consumers can be added or removed without impacting the brokers or other consumers. If a consumer fails, its workload can be redistributed to other consumers in the group without losing data, thanks to the consumer's control over message fetching and offset management.


### Start Kafka server
```bash
docker-compose-up
```

### Create a consumer that listens to a topic
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my.first.topic
```

### Create a consumer that listens to a topic
```bash
docker run -it --network host --rm bitnami/kafka:3.6.1 kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my.first.topic
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
