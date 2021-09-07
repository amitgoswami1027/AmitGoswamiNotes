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


