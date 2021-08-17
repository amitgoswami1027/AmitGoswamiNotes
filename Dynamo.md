# Dynamo: How To Design a Key-Value Store


## Goal : Design a distributed key-value store that is highly available (i.e., reliable), highly scalable, and completely decentralized.

## What is Dynamo?
Dynamo is a highly available key-value store developed by Amazon for their internal use. Many Amazon services, such as shopping cart, bestseller lists, sales rank, product catalog, etc., need only primary-key access to data. A multi-table relational database system would be an overkill for such services and would also limit scalability and availability. Dynamo provides a flexible design to let applications choose their desired level of availability and consistency.

## Background
Dynamo – not to be confused with DynamoDB, which was inspired by Dynamo’s design – is a distributed key-value storage system that provides an “always-on” (or highly available) experience at a massive scale. In CAP theorem terms, Dynamo falls within the category of AP systems (i.e., available and partition tolerant) and is designed for high availability and partition tolerance at the expense of strong consistency. The primary motivation for designing Dynamo as a highly available system was the observation that the availability of a system directly correlates to the number of customers served. Therefore, the main goal is that the system, even when it is imperfect, should be available to the customer as it brings more customer satisfaction. On the other hand, inconsistencies can be resolved in the background, and most of the time they will not be noticeable by the customer. Derived from this core principle, Dynamo is aggressively optimized for availability.

The Dynamo design was highly influential as it inspired many NoSQL databases, like Cassandra, Riak, and Voldemort – not to mention Amazon’s own DynamoDB.

## Design goals
As stated above, the main goal of Dynamo is to be highly available. Here is the summary of its other design goals:

Scalable: The system should be highly scalable. We should be able to throw a machine into the system to see proportional improvement.
Decentralized: To avoid single points of failure and performance bottlenecks, there should not be any central/leader process.
Eventually Consistent: Data can be optimistically replicated to become eventually consistent. This means that instead of incurring write-time costs to ensure data correctness throughout the system (i.e., strong consistency), inconsistencies can be resolved at some other time (e.g., during reads). Eventual consistency is used to achieve high availability.


![image](https://user-images.githubusercontent.com/13011167/129662887-bf513253-670a-41ac-9175-c88e4dd5d282.png)

## Dynamo’s use cases
By default, Dynamo is an eventually consistent database. Therefore, any application where strong consistency is not a concern can utilize Dynamo. Though Dynamo can support strong consistency, it comes with a performance impact. Hence, if strong consistency is a requirement for an application, then Dynamo might not be a good option.

Dynamo is used at Amazon to manage services that have very high-reliability requirements and need tight control over the trade-offs between availability, consistency, cost-effectiveness, and performance. Amazon’s platform has a very diverse set of applications with different storage requirements. Many applications chose Dynamo because of its flexibility for selecting the appropriate trade-offs to achieve high availability and guaranteed performance in the most cost-effective manner.

Many services on Amazon’s platform require only primary-key access to a data store. For such services, the common pattern of using a relational database would lead to inefficiencies and limit scalability and availability. Dynamo provides a simple primary-key only interface to meet the requirements of these applications.

## System APIs 
The Dynamo clients use put() and get() operations to write and read data corresponding to a specified key. This key uniquely identifies an object.

    get(key): The get operation finds the nodes where the object associated with the given key is located and returns either a single object or a list of objects with conflicting versions along with a context. The context contains encoded metadata about the object that is meaningless to the caller and includes information such as the version of the object (more on this below).

    put(key, context, object): The put operation finds the nodes where the object associated with the given key should be stored and writes the given object to the disk. The context is a value that is returned with a get operation and then sent back with the put operation. The context is always stored along with the object and is used like a cookie to verify the validity of the object supplied in the put request.

Dynamo treats both the object and the key as an arbitrary array of bytes (typically less than 1 MB). It applies the MD5 hashing algorithm on the key to generate a 128-bit identifier which is used to determine the storage nodes that are responsible for serving the key.

# Dynamo’s architecture
At a high level, Dynamo is a Distributed Hash Table (DHT) that is replicated across the cluster for high availability and fault tolerance. Dynamo’s architecture can be summarized as - Data distribution, Data replication and consistency, Handling temporary failures, Inter-node communication and failure detection, High availability, Conflict resolution and handling permanent failures.

## What is data partitioning
The act of distributing data across a set of nodes is called data partitioning. There are two challenges when we try to distribute data:

1. How do we know on which node a particular piece of data will be stored?
2. When we add or remove nodes, how do we know what data will be moved from existing nodes to the new nodes? Furthermore, how can we minimize data movement when nodes join or leave?

A naive approach will be to use a suitable hash function that maps the data key to a number. Then, find the server by applying modulo on this number and the total number of servers. The scheme described in the above diagram solves the problem of finding a server for storing/retrieving the data. But when we add or remove a server, we have to remap all the keys and move the data based on the new server count, which will be a complete mess!

Dynamo uses consistent hashing to solve these problems. The consistent hashing algorithm helps Dynamo map rows to physical nodes and also ensures that only a small set of keys move when servers are added or removed.

![image](https://user-images.githubusercontent.com/13011167/129663266-b15518a5-954f-43a5-9c94-5ae38170b7a7.png)

## Consistent hashing: Dynamo’s data distribution
Consistent hashing represents the data managed by a cluster as a ring. Each node in the ring is assigned a range of data. Dynamo uses the consistent hashing algorithm to determine what row is stored to what node. Here is an example of the consistent hashing ring:

![image](https://user-images.githubusercontent.com/13011167/129663348-2dc39189-ee2c-462a-8279-e9477389d132.png)

With consistent hashing, the ring is divided into smaller predefined ranges. Each node is assigned one of these ranges. In Dynamo’s terminology, the start of the range is called a token. This means that each node will be assigned one token. The range assigned to each node is computed as follows:

Range start:  Token value
Range end:    Next token value - 1

Here are the tokens and data ranges of the four nodes described in the above diagram:
![image](https://user-images.githubusercontent.com/13011167/129663405-b83261ac-6898-4c8d-bbfe-fecce5561e80.png)

Whenever Dynamo is serving a put() or a get() request, the first step it performs is to apply the MD5 hashing algorithm to the key. The output of this hashing algorithm determines within which range the data lies and hence, on which node the data will be stored. As we saw above, each node in Dynamo is supposed to store data for a fixed range. Hence, the hash generated from the data key tells us the node where the data will be stored. Here is an example showing how data gets distributed across the Consistent Hashing ring:

![image](https://user-images.githubusercontent.com/13011167/129663471-5fd9d28f-9bf6-414b-afa8-bf54425d91be.png)

The consistent hashing scheme described above works great when a node is added or removed from the ring; as only the next node is affected in these scenarios. For example, when a node is removed, the next node becomes responsible for all of the keys stored on the outgoing node. However, this scheme can result in non-uniform data and load distribution. Dynamo solves these issues with the help of Virtual nodes.

## Virtual nodes
Adding and removing nodes in any distributed system is quite common. Existing nodes can die and may need to be decommissioned. Similarly, new nodes may be added to an existing cluster to meet growing demands. Dynamo efficiently handles these scenarios through the use of virtual nodes (or Vnodes).

As we saw above, the basic Consistent Hashing algorithm assigns a single token (or a consecutive hash range) to each physical node. This was a static division of ranges that requires calculating tokens based on a given number of nodes. This scheme made adding or replacing a node an expensive operation, as, in this case, we would like to rebalance and distribute the data to all other nodes, resulting in moving a lot of data. Here are a few potential issues associated with a manual and fixed division of the ranges:

    Adding or removing nodes: Adding or removing nodes will result in recomputing the tokens causing a significant administrative overhead for a large cluster.
    Hotspots: Since each node is assigned one large range, if the data is not evenly distributed, some nodes can become hotspots.
    Node rebuilding: Since each node’s data is replicated on a fixed number of nodes (discussed later), when we need to rebuild a node, only its replica nodes can provide the data. This puts a lot of pressure on the replica nodes and can lead to service degradation.

To handle these issues, Dynamo introduced a new scheme for distributing the tokens to physical nodes. Instead of assigning a single token to a node, the hash range is divided into multiple smaller ranges, and each physical node is assigned multiple of these smaller ranges. Each of these subranges is called a Vnode. With Vnodes, instead of a node being responsible for just one token, it is responsible for many tokens (or subranges).

![image](https://user-images.githubusercontent.com/13011167/129663610-4e59ce49-b098-4c32-95c3-85746304b4c7.png)

Practically, Vnodes are randomly distributed across the cluster and are generally non-contiguous so that no two neighboring Vnodes are assigned to the same physical node. Furthermore, nodes do carry replicas of other nodes for fault-tolerance. Also, since there can be heterogeneous machines in the clusters, some servers might hold more Vnodes than others. The figure below shows how physical nodes A, B, C, D, & E are using Vnodes of the Consistent Hash ring. Each physical node is assigned a set of Vnodes and each Vnode is replicated once.

![image](https://user-images.githubusercontent.com/13011167/129663663-75df2094-53ed-44be-9d8e-a0f11a368dee.png)

### Advantages of Vnodes 
Vnodes give the following advantages:

* Vnodes help spread the load more evenly across the physical nodes on the cluster by dividing the hash ranges into smaller subranges. This speeds up the rebalancing process after adding or removing nodes. When a new node is added, it receives many Vnodes from the existing nodes to maintain a balanced cluster. Similarly, when a node needs to be rebuilt, instead of getting data from a fixed number of replicas, many nodes participate in the rebuild process.
* Vnodes make it easier to maintain a cluster containing heterogeneous machines. This means, with Vnodes, we can assign a high number of ranges to a powerful server and a lower number of ranges to a less powerful server.
* Since Vnodes help assign smaller ranges to each physical node, the probability of hotspots is much less than the basic Consistent Hashing scheme which uses one big range per node.

## What is optimistic replication?
To ensure high availability and durability, Dynamo replicates each data item on multiple NNN nodes in the system where the value NNN is equivalent to the replication factor and is configurable per instance of Dynamo. Each key is assigned to a coordinator node (the node that falls first in the hash range), which first stores the data locally and then replicates it to N−1N-1N−1 clockwise successor nodes on the ring. This results in each node owning the region on the ring between it and its NthNthNth predecessor. This replication is done asynchronously (in the background), and Dynamo provides an eventually consistent model. This replication technique is called optimistic replication, which means that replicas are not guaranteed to be identical at all times.

![image](https://user-images.githubusercontent.com/13011167/129663793-b937871e-bed9-44f9-941b-10939dadaba0.png)

Each node in Dynamo serves as a replica for a different range of data. As Dynamo stores NNN copies of data spread across different nodes, if one node is down, other replicas can respond to queries for that range of data. If a client cannot contact the coordinator node, it sends the request to a node holding a replica.

## Preference List 
The list of nodes responsible for storing a particular key is called the preference list. Dynamo is designed so that every node in the system can determine which nodes should be in this list for any specific key (discussed later). This list contains more than NNN nodes to account for failure and skip virtual nodes on the ring so that the list only contains distinct physical nodes.





