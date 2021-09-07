# Kafka Messaging System

## GOAL
Design a distributed messaging system that can reliably transfer a high throughput of messages between different entities. One of the common challenges among distributed systems is handling a continuous influx of data from multiple sources. Imagine a log aggregation service that is receiving hundreds of log entries per second from different sources. The function of this log aggregation service is to store these logs on disk at a shared server and also build an index so that the logs can be searched later. A few challenges of this service are:
* How will the log aggregation service handle a spike of messages? If the service can handle (or buffer) 500 messages per second, what will happen if it starts receiving a higher number of messages per second? If we decide to have multiple instances of the log aggregation service, how do we divide the work among these instances?
* How can we receive messages from different types of sources? The sources producing (or consuming) these logs need to decide upon a common protocol and data format to send log messages to the log aggregation service. This leads us to a strongly coupled architecture between the producer and consumer of the log messages.
* What will happen to the log messages if the log aggregation service is down or unresponsive for some time?

### What is a messaging system?#
A messaging system is responsible for transferring data among services, applications, processes, or servers. Such a system helps decouple different parts of a distributed system by providing an asynchronous way of transferring messaging between the sender and the receiver. Hence, all senders (or producers) and receivers (or consumers) focus on the data/message without worrying about the mechanism used to share the data.
![image](https://user-images.githubusercontent.com/13011167/132287114-d2129b52-ae5d-4f12-bbdc-7a4073ea1b1d.png)

There are two common ways to handle messages: Queuing and Publish-Subscribe.
### QUEUE : In the queuing model, messages are stored sequentially in a queue. Producers push messages to the rear of the queue, and consumers extract the messages from the front of the queue.A particular message can be consumed by a maximum of one consumer only. Once a consumer grabs a message, it is removed from the queue such that the next consumer will get the next message. This is a great model for distributing message-processing among multiple consumers. But this also limits the system as multiple consumers cannot read the same message from the queue.

![image](https://user-images.githubusercontent.com/13011167/132287192-2dd60523-712f-4843-b2cf-15481540f687.png)

### Publish-subscribe messaging system : In the pub-sub (short for publish-subscribe) model, messages are divided into topics. A publisher (or a producer) sends a message to a topic that gets stored in the messaging system under that topic. Subscribers (or the consumer) subscribe to a topic to receive every message published to that topic. Unlike the Queuing model, the pub-sub model allows multiple consumers to get the same message; if two consumers subscribe to the same topic, they will receive all messages published to that topic.


* The messaging system that stores and maintains the messages is commonly known as the message broker. It provides a loose coupling between publishers and subscribers, or producers and consumers of data. (Providing abstraction)
* The message broker stores published messages in a queue, and subscribers read them from the queue. Hence, subscribers and publishers do not have to be synchronized. This loose coupling enables subscribers and publishers to read and write messages at different rates.(Messaging buffering)
* The messaging system’s ability to store messages provides fault-tolerance, so messages do not get lost between the time they are produced and the time they are consumed. (Guarantee of message delivery. )

![image](https://user-images.githubusercontent.com/13011167/132287305-89b6aab5-d2d2-49f5-b985-64b008c421f7.png)

## WHAT IS KAFKA ?
Apache Kafka is an open-source publish-subscribe-based messaging system (Kafka can work as a message queue too, more on this later). It is distributed, durable, fault-tolerant, and highly scalable by design. Fundamentally, it is a system that takes streams of messages from applications known as producers, stores them reliably on a central cluster (containing a set of brokers), and allows those messages to be received by applications (known as consumers) that process the messages.
![image](https://user-images.githubusercontent.com/13011167/132287683-f321368d-3287-47d9-9771-6f24beff7fba.png)

### BACKGROUND
Kafka was created at LinkedIn around 2010 to track various events, such as page views, messages from the messaging system, and logs from various services. Later, it was made open-source and developed into a comprehensive system which is used for:
* Reliably storing a huge amount of data.
* Enabling high throughput of message transfer between different entities.
* Streaming real-time data.

At a high level, we can call Kafka a distributed Commit Log. A Commit Log (also known as a Write-Ahead log or a Transactions log) is an append-only data structure that can persistently store a sequence of records. Records are always appended to the end of the log, and once added, records cannot be deleted or modified. Reading from a commit log always happens from left to right (or old to new). 

* Kafka stores all of its messages on disk. Since all reads and writes happen in sequence, Kafka takes advantage of sequential disk reads (more on this later).

![image](https://user-images.githubusercontent.com/13011167/132287823-87b1aec3-d4ed-4b21-8c82-c7b863016ab1.png)

# Kafka use cases
* Metrics: Kafka can be used to collect and aggregate monitoring data. Distributed services can push different operational metrics to Kafka servers. These metrics can then be pulled from Kafka to produce aggregated statistics.
* Log Aggregation: Kafka can be used to collect logs from multiple sources and make them available in a standard format to multiple consumers.
* Stream processing: Kafka is quite useful for use cases where the collected data undergoes processing at multiple stages. For example, the raw data consumed from a topic is transformed, enriched, or aggregated and pushed to a new topic for further consumption. This way of data processing is known as stream processing
* Commit Log: Kafka can be used as an external commit log for any distributed system. Distributed services can log their transactions to Kafka to keep track of what is happening. This transaction data can be used for replication between nodes and also becomes very useful for disaster recovery, for example, to help failed nodes to recover their states.
* Website activity tracking: One of Kafka’s original use cases was to build a user activity tracking pipeline. User activities like page clicks, searches, etc., are published to Kafka into separate topics. These topics are available for subscription for a range of use cases, including real-time processing, real-time monitoring, or loading into Hadoop or data warehousing systems for offline processing and reporting.
* Product suggestions: Imagine an online shopping site like amazon.com, which offers a feature of ‘similar products’ to suggest lookalike products that a customer could be interested in buying. To make this work, we can track every consumer action, like search queries, product clicks, time spent on any product, etc., and record these activities in Kafka. Then, a consumer application can read these messages to find correlated products that can be shown to the customer in real-time. Alternatively, since all data is persistent in Kafka, a batch job can run overnight on the ‘similar product’ information gathered by the system, generating an email for the customer with product suggestions.

## HIGH LEVEL ARCHITECTURE OF KAFKA
### Broker : A Kafka server is also called a broker. Brokers are responsible for reliably storing data provided by the producers and making it available to the consumers.
### Records : A record is a message or an event that gets stored in Kafka. Essentially, it is the data that travels from producer to consumer through Kafka. A record contains a key, a value, a timestamp, and optional metadata headers.
![image](https://user-images.githubusercontent.com/13011167/132288312-5ec2c129-0822-4205-acd2-cc5faec590bb.png)

### Topics
Kafka divides its messages into categories called Topics. In simple terms, a topic is like a table in a database, and the messages are the rows in that table. 
* Each message that Kafka receives from a producer is associated with a topic.
* Consumers can subscribe to a topic to get notified when new messages are added to that topic.
* A topic can have multiple subscribers that read messages from it.
* In a Kafka cluster, a topic is identified by its name and must be unique.

Messages in a topic can be read as often as needed — unlike traditional messaging systems, messages are not deleted after consumption. Instead, Kafka retains messages for a configurable amount of time or until a storage size is exceeded. Kafka’s performance is effectively constant with respect to data size, so storing data for a long time is perfectly fine.

![image](https://user-images.githubusercontent.com/13011167/132288585-287fc4ee-cfc1-4f38-bb51-897f4c555393.png)

### Kafka Cluster : Kafka is deployed as a cluster of one or more servers, where each server is responsible for running one Kafka broker
### ZooKeeper
ZooKeeper is a distributed key-value store and is used for coordination and storing configurations. It is highly optimized for reads. Kafka uses ZooKeeper to coordinate between Kafka brokers; ZooKeeper maintains metadata information about the Kafka cluster. 

![image](https://user-images.githubusercontent.com/13011167/132288836-a4f91634-ac28-405c-8ed3-cf2d8282ff64.png)

## KAFKA DEEP DIVE
* Kafka is simply a collection of topics. As topics can get quite big, they are split into partitions of a smaller size for better performance and scalability. 
* Kafka topics are partitioned, meaning a topic is spread over a number of ‘fragments’. Each partition can be placed on a separate Kafka broker. When a new message is published on a topic, it gets appended to one of the topic’s partitions. The producer controls which partition it publishes messages to based on the data. For example, a producer can decide that all messages related to a particular ‘city’ go to the same partition.
* Essentially, a partition is an ordered sequence of messages. Producers continually append new messages to partitions. Kafka guarantees that all messages inside a partition are stored in the sequence they came in. Ordering of messages is maintained at the partition level, not across the topic.

![image](https://user-images.githubusercontent.com/13011167/132289200-88a1cc66-c22c-4cae-90d5-f610b905dd72.png)

* A unique sequence ID called an offset gets assigned to every message that enters a partition. These numerical offsets are used to identify every message’s sequential position within a topic’s partition.
* Offset sequences are unique only to each partition. This means, to locate a specific message, we need to know the Topic, Partition, and Offset number.
* Producers can choose to publish a message to any partition. If ordering within a partition is not needed, a round-robin partition strategy can be used, so records get distributed evenly across partitions.
* Placing each partition on separate Kafka brokers enables multiple consumers to read from a topic in parallel. That means, different consumers can concurrently read different partitions present on separate brokers.
* Placing each partition of a topic on a separate broker also enables a topic to hold more data than the capacity of one server.
* Messages once written to partitions are immutable and cannot be updated.
* A producer can add a ‘key’ to any message it publishes. Kafka guarantees that messages with the same key are written to the same partition.
* Each broker manages a set of partitions belonging to different topics.

Kafka follows the principle of a dumb broker and smart consumer. This means that Kafka does not keep track of what records are read by the consumer.Every topic can be replicated to multiple Kafka brokers to make the data fault-tolerant and highly available. Each topic partition has one leader broker and multiple replica (follower) brokers. Instead, consumers, themselves, poll Kafka for new messages and say what records they want to read. This allows them to increment/decrement the offset they are at as they wish, thus being able to replay and reprocess messages. Consumers can read messages starting from a specific offset and are allowed to read from any offset they choose. This also enables consumers to join the cluster at any point in time.

### LEADER: A leader is the node responsible for all reads and writes for the given partition. Every partition has one Kafka broker acting as a leader.
### FOLLOWER: To handle single point of failure, Kafka can replicate partitions and distribute them across multiple broker servers called followers. Each follower’s responsibility is to replicate the leader’s data to serve as a ‘backup’ partition. This also means that any follower can take over the leadership if the leader goes down. 
### ZOOKEEPER: Kafka stores the location of the leader of each partition in ZooKeeper. As all writes/reads happen at/from the leader, producers and consumers directly talk to ZooKeeper to find a partition leader.

![image](https://user-images.githubusercontent.com/13011167/132289795-65d4a6c9-a8d3-434b-b7f1-7d44f92be021.png)
### HIGH WATER MARK
To ensure data consistency, the leader broker never returns (or exposes) messages which have not been replicated to a minimum set of ISRs. For this, brokers keep track of the high-water mark, which is the highest offset that all ISRs of a particular partition share. The leader exposes data only up to the high-water mark offset and propagates the high-water mark offset to all followers. Let’s understand this with an example

In the figure below, the leader does not return messages greater than offset ‘4’, as it is the highest offset message that has been replicated to all follower brokers.If a consumer reads the record with offset ‘7’ from the leader (Broker 1), and later, if the current leader fails, and one of the followers becomes the leader before the record is replicated to the followers, the consumer will not be able to find that message on the new leader. The client, in this case, will experience a non-repeatable read. Because of this possibility, Kafka brokers only return records up to the high-water mark.

![image](https://user-images.githubusercontent.com/13011167/132290009-6bcabfd0-7fd4-4e8a-8d59-5d22c081164c.png)

# What is a consumer group?
A consumer group is basically a set of one or more consumers working together in parallel to consume messages from topic partitions. Messages are equally divided among all the consumers of a group, with no two consumers receiving the same message.
## Distributing partitions to consumers within a consumer group
Kafka ensures that only a single consumer reads messages from any partition within a consumer group. In other words, topic partitions are a unit of parallelism – only one consumer can work on a partition in a consumer group at a time. If a consumer stops, Kafka spreads partitions across the remaining consumers in the same consumer group. Similarly, every time a consumer is added to or removed from a group, the consumption is rebalanced within the group.

![image](https://user-images.githubusercontent.com/13011167/132290494-e8793878-44bf-4c83-ae5e-15a239265eeb.png)

* Consumers pull messages from topic partitions. Different consumers can be responsible for different partitions. Kafka can support a large number of consumers and retain large amounts of data with very little overhead. By using consumer groups, consumers can be parallelized so that multiple consumers can read from multiple partitions on a topic, allowing a very high message processing throughput. The number of partitions impacts consumers’ maximum parallelism, as there cannot be more consumers than partitions.
* Kafka stores the current offset per consumer group per topic per partition, as it would for a single consumer. This means that unique messages are only sent to a single consumer in a consumer group, and the load is balanced across consumers as equally as possible.
* When the number of consumers exceeds the number of partitions in a topic, all new consumers wait in idle mode until an existing consumer unsubscribes from that partition. Similarly, as new consumers join a consumer group, Kafka initiates a rebalancing if there are more consumers than partitions. Kafka uses any unused consumers as failovers.

Here is a summary of how Kafka manages the distribution of partitions to consumers within a consumer group:

Number of consumers in a group = number of partitions: each consumer consumes one partition.
Number of consumers in a group > number of partitions: some consumers will be idle.
Number of consumers in a group < number of partitions: some consumers will consume more partitions than others.

# KAFKA WORKFLOW
## Kafka workflow as pub-sub messaging
* Producers publish messages on a topic.
* Kafka broker stores messages in the partitions configured for that particular topic. If the producer did not specify the partition in which the message should be stored, the broker ensures that the messages are equally shared between partitions. If the producer sends two messages and there are two partitions, Kafka will store one message in the first partition and the second message in the second partition.
* Consumer subscribes to a specific topic.
* Once the consumer subscribes to a topic, Kafka will provide the current offset of the topic to the consumer and also saves that offset in the ZooKeeper.
* Consumer will request Kafka at regular intervals for new messages.
* Once Kafka receives the messages from producers, it forwards these messages to the consumer.
* Consumer will receive the message and process it.
* Once the messages are processed, the consumer will send an acknowledgment to the Kafka broker.
* Upon receiving the acknowledgment, Kafka increments the offset and updates it in the ZooKeeper. Since offsets are maintained in the ZooKeeper, the consumer can read the next message correctly, even during broker outages.
* The above flow will repeat until the consumer stops sending the request.
* Consumers can rewind/skip to the desired offset of a topic at any time and read all the subsequent messages.

## Kafka workflow for consumer group
Instead of a single consumer, a group of consumers from one consumer group subscribes to a topic, and the messages are shared among them. Let us check the workflow of this system:
* Producers publish messages on a topic.
* Kafka stores all messages in the partitions configured for that particular topic, similar to the earlier scenario.
* A single consumer subscribes to a specific topic, assume Topic-01 with Group ID as Group-1.
* Kafka interacts with the consumer in the same way as pub-sub messaging until a new consumer subscribes to the same topic, Topic-01, with the same Group ID as Group-1.
* Once the new consumer arrives, Kafka switches its operation to share mode, such that each message is passed to only one of the subscribers of the consumer group Group-1. This message transfer is similar to queue-based messaging, as only one consumer of the group consumes a message. Contrary to queue-based messaging, messages are not removed after consumption.
* This message transfer can go on until the number of consumers reaches the number of partitions configured for that particular topic.
* Once the number of consumers exceeds the number of partitions, the new consumer will not receive any message until an existing consumer unsubscribes. This scenario arises because each consumer in Kafka will be assigned a minimum of one partition. Once all the partitions are assigned to the existing consumers, the new consumers will have to wait.

# ROLE OF ZOOKEEPER
A critical dependency of Apache Kafka is Apache ZooKeeper, which is a distributed configuration and synchronization service. ZooKeeper serves as the coordination interface between the Kafka brokers, producers, and consumers. Kafka stores basic metadata in ZooKeeper, such as information about brokers, topics, partitions, partition leader/followers, consumer offsets, etc.
![image](https://user-images.githubusercontent.com/13011167/132291496-2d3acaaa-71ef-4725-86c9-2b6d007961d7.png)

### ZooKeeper as the central coordinator#
As we know, Kafka brokers are stateless; they rely on ZooKeeper to maintain and coordinate brokers, such as notifying consumers and producers of the arrival of a new broker or failure of an existing broker, as well as routing all requests to partition leaders. ZooKeeper is used for storing all sorts of metadata about the Kafka cluster:
* It maintains the last offset position of each consumer group per partition, so that consumers can quickly recover from the last position in case of a failure (although modern clients store offsets in a separate Kafka topic).
* It tracks the topics, number of partitions assigned to those topics, and leaders’/followers’ location in each partition.
* It also manages the access control lists (ACLs) to different topics in the cluster. ACLs are used to enforce access or authorization.

### How do producers or consumers find out who the leader of a partition is?


