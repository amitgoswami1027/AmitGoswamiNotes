# Distributed Google File System !!

Design a distributed file system to store huge files (terabyte and larger). The system should be scalable, reliable, and highly available. GFS is a distributed file system built for large, distributed data-intensive applications like Gmail or YouTube.oogle’s BigTable uses the distributed Google File System to store log and data files. 

## APIs
GFS does not provide standard POSIX-like APIs; instead, user-level APIs are provided. In GFS, files are organized hierarchically in directories and identified by their pathnames. GFS supports the usual file system operations:

* create – To create a new instance of a file.
* delete – To delete an instance of a file.
* open – To open a named file and return a handle.
* close – To close a given file specified by a handle.
* read – To read data from a specified file and offset.
* write – To write data to a specified file and offset.

In addition, GFS supports two special operations:
* Snapshot: A snapshot is an efficient way of creating a copy of the current instance of a file or directory tree.
* Append: An append operation allows multiple clients to append data to the same file concurrently while guaranteeing atomicity. It is useful for implementing multi-way merge results and producer-consumer queues that many clients can simultaneously append to without additional locking.

## High-level Architecture

### CHUNKS 
A GFS cluster consists of a single master and multiple ChunkServers and is accessed by multiple clients. As files stored in GFS tend to be very large, GFS breaks files into multiple fixed-size chunks where each chunk is 64 megabytes in size. Each chunk is identified by an immutable and globally unique 64-bit ID number called chunk handle. This allows for 2^{64} unique chunks. If each chunk is 64 MB, total storage space would be more than 10^9 exa-bytes. As files are split into chunks, therefore, the job of GFS is to provide a mapping from files to chunks, and then to support standard operations on files, mapping down to operations on individual chunks.

### CLUSTER
GFS is organized into a simple network of computers called a cluster. All GFS clusters contain three kinds of entities:
* A single master server
* Multiple ChunkServers
* Many clients
The master stores all metadata about the system, while the ChunkServers store the real file data.

![image](https://user-images.githubusercontent.com/13011167/132126373-e426c4d1-f3de-4be7-97c9-f54cd81eb7b0.png)

ChunkServers store chunks on local disks as regular Linux files and read or write chunk data specified by a chunk handle and byte-range. For reliability, each chunk is replicated to multiple ChunkServers. By default, GFS stores three replicas, though different replication factors can be specified on a per-file basis.
![image](https://user-images.githubusercontent.com/13011167/132126438-a3360116-a01b-4127-99a2-2e80b9ee0906.png)

### Master
* Master server is the coordinator of a GFS cluster and is responsible for keeping track of filesystem metadata.
* The master also controls system-wide activities such as chunk lease management (locks on chunks with expiration), garbage collection of orphaned chunks, and chunk migration between ChunkServers. Master assigns chunk handle to chunks at time of chunk creation.
* For performance and fast random access, all metadata is stored in the master’s main memory. This includes the entire filesystem namespace as well as all the name-to-chunk mappings.
* For fault tolerance and to handle a master crash, all metadata changes are written to the disk onto an operation log. This operation log is also replicated onto remote machines. The operation log is similar to a journal. Every operation to the file system is logged into this file
* The master is a single point of failure, hence, it replicates its data onto several remote machines so that the master can be readily restored on failure.
* Having a single master vastly simplifies GFS design and enables the master to make sophisticated chunk placement and replication decisions using global knowledge. However, GFS minimizes the master’s involvement in reads and writes so that it does not become a bottleneck. Clients never read or write file data through the master. Instead, a client asks the master which ChunkServers it should contact. The client caches this information for a limited time and interacts with the ChunkServers directly for many subsequent operations.




