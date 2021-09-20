# CHUBBY - DISTRIBUTED LOCKING SERVICE
# Optimistic vs. Pessimistic Locking

## Pessimistic Locking - Pessimistic Lock is where you assume that all the users are trying to access the same record and it literally locks the record exclusively for the first started transaction until it is completed successfully or failed. Then the lock is released and the next transaction on the record is handled in the same way.Pessimistic Lock is where you assume that all the users are trying to access the same record and it literally locks the record exclusively for the first started transaction until it is completed successfully or failed. Then the lock is released and the next transaction on the record is handled in the same way.
* In our case, if we apply Pessimistic Lock, the first user to come and buy the last item in the stock will click on “Purchase”.
* This will lock the object until the payment is completed or failed.
* Let’s say User A was able to pay for it and the stock value for that item is set to 0 now.
* All the other users have to wait during this process.
* Now all the other users will see that the item went out of stocks, and cannot do anything with the item.

## Optimistic Lock - Optimistic Lock is where you manage your data by checking a special value in the database — it is often a version number, timestamp, date, etc.— before you read/write to the data to make sure that the data you are dealing with is not stale/old/changed since you’ve last viewed. If the data is stale, the transaction is not completed successfully and an error is thrown to indicate that. Something like: “The record you attempted to edit was modified by another user after you got the original value”.
* In our case, if we apply Optimistic Lock, the first user to come and buy the last item in the stock will click on “Purchase”.
* Let’s say User A was able to pay for it and before the payment step the stock value is checked before committing to change it from 1 to 0.
* If the version numbers match the operation is committed and the item goes out of stock.
* Now all the other users that try to purchase that the item will be warned about that the item is no longer available, right at the moment they try to buy, pay or add it to their baskets.

Please note that:
* Optimistic locking is just a mechanism to prevent processes from overwriting changes by another process. Optimistic locking is not a magic wand to manage or auto-merge any conflicting changes. It can only allow users to alert or notify about such conflicting changes.
* Optimistic locking works by just comparing the value of the “version” column. Thus, optimistic locking is not a real database lock.

## SUMMARY
Both pessimistic and optimistic locking mechanisms have use cases that they fit in, however, for systems with high loads of updates which hold locks for a significant time interval, the optimistic shows better efficiency. Also, it must be noted that the higher RPS doesn’t necessarily means a faster algorithm, since an important amount of requests are failed because of the lock-wait errors in the pessimistic mechanism.

# DISTRIBUTED LOCKING MECHANISM (CHUBBY) 
Goal - Design a highly available and consistent service that can store small objects and provide a locking mechanism on those objects. Chubby is a service that provides a distributed locking mechanism and also stores small files.Primarily Chubby was developed to provide a reliable locking service. Over time, some interesting uses of Chubby have evolved. Following are the top use cases where Chubby is practically being used:
* Leader/master election
* Naming service (like DNS)
* Storage (small objects that rarely change)
* Distributed locking mechanism

Chubby is used extensively inside Google in various systems such as GFS, BigTable. The primary goal is to provide a reliable lock service. Chubby is NOT optimized for high performance, frequent locking scenarios. There are thousands of clients that use Chubby, but they use Chubby occasionally — for coarse-grained locking. Generally such coarse grained locks are held for hours or days and NOT seconds. One typical use of Chubby is for multiple applications is to elect a master — the first one getting the lock wins and becomes the master.

## DESIGN DECISION
Following main design decisions come out from the topics mentioned in the last section.
* Coarse grained locking — Applications don’t need locks of shorter duration. For example, electing a master is not a frequent event.
* Small data storing(small file manipulations)capabilities in addition to a lock service
* Allow thousands of clients to observe changes. So lock service needs to scale to handle many clients, although the transaction rate may not be that hi
* Notification mechanism by which client knows when the change occurs in the file that is shared e.g. when a primary changes
* Support client side caching to deal with clients that may want to poll aggressivel
* Strong caching guarantees to simplify developer usage

### Leader/master election
Any lock service can be seen as a consensus service, as it converts the problem of reaching consensus to handing out locks. Basically, a set of distributed applications compete to acquire a lock, and whoever gets the lock first gets the resource. Similarly, an application can have multiple replicas running and wants one of them to be chosen as the leader. Chubby can be used for leader election among a set of replicas, e.g., the leader/master for GFS and BigTable.
![image](https://user-images.githubusercontent.com/13011167/133948373-b3366f25-7fcb-4c45-ae65-a61fe4d7aeb6.png)

### Naming service (like DNS)
It is hard to make faster updates to DNS due to its time-based caching nature, which means there is generally a potential delay before the latest DNS mapping is effective. As a result, chubby has replaced DNS inside Google as the main way to discover servers.
![image](https://user-images.githubusercontent.com/13011167/133948409-8127ffa7-fda9-409f-8345-3c538384bed5.png)

### Storage (small objects that rarely change)
Chubby provides a Unix-style interface to reliably store small files that do not change frequently (complementing the service offered by GFS). Applications can then use these files for any usage like DNS, configs, etc. GFS and Bigtable store their metadata in Chubby. Some services use Chubby to store ACL files.

![image](https://user-images.githubusercontent.com/13011167/133948452-30e13b36-f112-42a3-9426-ec3640fad6d2.png)

### Distributed locking mechanism
Chubby provides a developer-friendly interface for coarse-grained distributed locks (as opposed to fine-grained locks) to synchronize distributed activities in a distributed environment. All an application needs is a few code lines, and Chubby service takes care of all the lock management so that developers can focus on application business logic. In other words, we can say that Chubby provides mechanisms like semaphores and mutexes for a distributed environment.
![image](https://user-images.githubusercontent.com/13011167/133948511-024a9d2c-7c21-4253-a1a8-c5ee0b9e631c.png)

### When not to use Chubby
Because of its design choices and proposed usage, Chubby should not be used when:
* Bulk storage is needed.
* Data update rate is high.
* Locks are acquired/released frequently.
* Usage is more like a publish/subscribe model.

## Chubby and Paxos
Paxos plays a major role inside Chubby. Readers familiar with Distributed Computing recognize that getting all nodes in a distributed system to agree on anything (e.g., election of primary among peers) is basically a kind of distributed consensus problem. Distributed consensus using Asynchronous Communication is already solved by Paxos protocol, and Chubby actually uses Paxos underneath to manage the state of the Chubby system at any point in time.

![image](https://user-images.githubusercontent.com/13011167/133948637-fe58d1b9-2272-47aa-96da-b81ee39c37f0.png)


# High-level Architecture
### Chubby cell
A Chubby Cell basically refers to a Chubby cluster. Most Chubby cells are confined to a single data center or machine room, though there can be a Chubby cell whose replicas are separated by thousands of kilometers. A single Chubby cell has two main components, server and client, that communicate via remote procedure call (RPC).

### Chubby servers
* A chubby cell consists of a small set of servers (typically 5) known as replicas.
* Using Paxos, one of the servers is chosen as the master who handles all client requests. If the master fails, another server from replicas becomes the master.
* Each replica maintains a small database to store files/directories/locks. The master writes directly to its own local database, which gets synced asynchronously to all the replicas. That’s how Chubby ensures data reliability and a smooth experience for clients even if the master fails.
* For fault tolerance, Chubby replicas are placed on different racks.

### Chubby client library
Client applications use a Chubby library to communicate with the replicas in the chubby cell using RPC.
![image](https://user-images.githubusercontent.com/13011167/133948801-ecbf4774-9a5d-4f3f-861e-71588b6fea47.png)

### Chubby APIs
* Chubby exports a file system interface similar to POSIX but simpler. It consists of a strict tree of files and directories in the usual way, with name components separated by slashes.
* File format: /ls/chubby_cell/directory_name/.../file_name
* We can divide Chubby APIs into following groups:

#### General
* Open(): Opens a given named file or directory and returns a handle.
* Close(): Closes an open handle.
* Poison(): Allows a client to cancel all Chubby calls made by other threads without fear of deallocating the memory being accessed by them.
* Delete(): Deletes the file or directory.

#### File
* GetContentsAndStat(): Returns (atomically) the whole file contents and metadata associated with the file. This approach of reading the whole file is designed to discourage the creation of large files, as it is not the intended use of Chubby.
* GetStat(): Returns just the metadata.
* ReadDir(): Returns the contents of a directory – that is, names and metadata of all children.
* SetContents(): Writes the whole contents of a file (atomically).
* SetACL(): Writes new access control list information

#### Locking
* Acquire(): Acquires a lock on a file.
* TryAquire(): Tries to acquire a lock on a file; it is a non-blocking variant of Acquire.
* Release(): Releases a lock.

#### Sequencer
* GetSequencer(): Get the sequencer of a lock. A sequencer is a string representation of a lock.
* SetSequencer(): Associate a sequencer with a handle.
* CheckSequencer(): Check whether a sequencer is valid.

Chubby does not support operations like append, seek, move files between directories, or making symbolic or hard links. Files can only be completely read or completely written/overwritten. This makes it practical only for storing very small files.

## Why was Chubby built as a service?
Let’s first understand the reason behind building a service instead of having a client library that only provides Paxos distributed consensus. A lock service has some clear advantages over a client library:
* Development becomes easy: Sometimes high availability is not planned in the early phases of development. Systems start as a prototype with little load and lose availability guarantees. As a service matures and gains more clients, availability becomes important; replication and primary election are then added to design. While this could be done with a library that provides distributed consensus, a lock server makes it easier to maintain the existing program structure and communication patterns. For example, electing a leader requires adding just a few lines of code. This technique is easier than making existing servers participate in a consensus protocol, especially if compatibility must be maintained during a transition period.
* Lock-based interface is developer-friendly: Programmers are generally familiar with locks. It is much easier to simply use a lock service in a distributed system than getting involved in managing Paxos protocol state locally, e.g., Acquire(), TryAcquire(), Release().
* Provide quorum & replica management: Distributed consensus algorithms need a quorum to make a decision, so several replicas are used for high availability. One can view the lock service as a way of providing a generic electorate that allows a client application to make decisions correctly when less than a majority of its own members are up. Without such support from a service, each application needs to have and manage its own quorum of servers.
* Broadcast functionality: Clients and replicas of a replicated service may wish to know when the service’s master changes; this requires an event notification mechanism. Such a mechanism is easy to build if there is a central service in the system.

## Why coarse-grained locks?
Chubby locks usage is not expected to be fine-grained in which they might be held for only a short period (i.e., seconds or less). For example, electing a leader is not a frequent event. Following are some main reasons why Chubby decided to only support coarse-grained locks:
* Less load on lock server: Coarse-grained locks impose far less load on the server as the lock acquisition rate is a lot less than the client’s transaction rate.
* Survive server failures: As coarse-grained locks are acquired rarely, clients are not significantly delayed by the temporary unavailability of the lock server. With fine-grained locks, even a brief unavailability of a lock server would cause many clients to stall.
* Fewer lock servers are needed: Coarse-grained locks allow many clients to be adequately served by a modest number of lock servers with somewhat lower availability.

## Chubby in terms of CAP theorem
As proved by CAP theorem, no application can be highly available, strongly consistent, and partition tolerant at the same time. Since Chubby guarantees strong consistency by ensuring that all read and write requests go through the master, technically, we can categories Chubby as a CP system.

Chubby’s architecture is based on a master-backup design which ensures strong consistency. If the master dies, Chubyy chooses a new master from the backup servers. Chubby continues to operate as long as no more than half of the servers fail, and it is guaranteed to make progress whenever the network is reliable. If the Chubby servers experience network partitioning, the service will become unavailable.

On the other hand, Chubby is optimized for the case where there is a stable master server, and there are no partitions. In this case, it delivers a very high degree of availability. So, overall, Chubby provides guaranteed consistency and a very high level of availability in the common case.

# HOW CHUBBY WORKS?
* Service initialization - Upon initialization, Chubby performs the following steps:
  * A master is chosen among Chubby replicas using Paxos.
  * Current master information is persisted in storage, and all replicas become aware of the master.

* Client initialization - Upon initialization, a Chubby client performs the following steps:
  * Client contacts the DNS to know the listed Chubby replicas.
  * Client calls any Chubby server directly via Remote Procedure Call (RPC).
  * If that replica is not the master, it will return the address of the current master.
  * Once the master is located, the client maintains a session with it and sends all requests to it until it indicates that it is not the master anymore or stops responding.

![image](https://user-images.githubusercontent.com/13011167/133949610-97e46767-7145-4b53-a31e-3959d03fa798.png)

## Leader election example using Chubby
Let’s take an example of an application that uses Chubby to elect a single master from a bunch of instances of the same application.

Once the master election starts, all candidates attempt to acquire a Chubby lock on a file associated with the election. Whoever acquires the lock first becomes the master. The master writes its identity on the file, so that other processes know who the is current master.

![image](https://user-images.githubusercontent.com/13011167/133949675-98538c93-1b20-4a44-b551-6dd26175890f.png)

### Sample pseudocode for leader election#
![image](https://user-images.githubusercontent.com/13011167/133949709-6ccef1d5-ab86-4e0c-aebc-61c2ce6c74e9.png)

# File, Directories, and Handles
Chubby file system interface is basically a tree of files and directories, where each directory contains a list of child files and directories. Each file or directory is called a node.

![image](https://user-images.githubusercontent.com/13011167/133949812-99005726-e3d0-4bc1-9c9b-751d78b1acbb.png)

### Metadata
Metadata for each node includes Access Control Lists (ACLs), four monotonically increasing 64-bit numbers, and a checksum
ACLs are used to control reading, writing, and changing the ACL names for the node.
* Node inherits the ACL names of its parent directory on creation.
* ACLs themselves are files located in an ACL directory, which is a well-known part of the cell’s local namespace.
* Users are authenticated by a mechanism built into the RPC system.

Monotonically increasing 64-bit numbers: These numbers allow clients to detect changes easily.
* An instance number: This is greater than the instance number of any previous node with the same name.
* A content generation number (files only): This is incremented every time a file’s contents are written.
* A lock generation number: This is incremented when the node’s lock transitions from free to held.
* An ACL generation number: This is incremented when the node’s ACL names are written.

Checksum: Chubby exposes a 64-bit file-content checksum so clients may tell whether files differ.

## Locks, Sequencers, and Lock-delays
