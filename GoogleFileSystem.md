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

# SINGLE MASTER
Having a single master vastly simplifies GFS design and enables the master to make sophisticated chunk placement and replication decisions using global knowledge. However, GFS minimizes the master’s involvement in reads and writes so that it does not become a bottleneck. Clients never read or write file data through the master. Instead, a client asks the master which ChunkServers it should contact. The client caches this information for a limited time and interacts with the ChunkServers directly for many subsequent operations.

![image](https://user-images.githubusercontent.com/13011167/132306823-02185572-d9e5-4ac4-85c2-acd6fc58c72c.png)

### Chunk size
Chunk size is one of the key design parameters. GFS has chosen 64 MB, which is much larger than typical filesystem block sizes (which are often around 4KB). Here are the advantages of using a large chunk size:
* Since GFS was designed to handle huge files, small chunk sizes would not make much sense, as each file would have a map of a huge number of chunks.
* As the master holds the metadata and manages file distribution, it is involved whenever chunks are read, modified, or deleted. This also means that a small chunk size would significantly increase the amount of data a master would need to manage and increase the amount of data that would need to be communicated to a client, resulting in extra network traffic.
* A large chunk size reduces the size of the metadata stored on the master, which enables the master to keep all the metadata in memory, thus significantly decreasing the latency for control operations.
*  By using a large chunk size, GFS reduces the need for frequent communication with the master to get chunk location information. It becomes feasible for a client to cache all information related to chunk locations of a large file. Client metadata caches have timeouts to reduce the risk of caching stale data.
*  A large chunk size also makes it possible to keep a TCP connection open to a ChunkServer for an extended time, amortizing the time of setting up a TCP connection.
*  A large chunk size simplifies ChunkServer management, i.e., to check which ChunkServers are near capacity or which are overloaded.
*  Large chunk size provides highly efficient sequential reads and appends of large amounts of data.

### Lazy space allocation
Each chunk replica is stored as a plain Linux file on a ChunkServer. GFS does not allocate the whole 64MB of disk space when creating a chunk. Instead, as the client appends data, the ChunkServer lazily extends the chunk. This lazy space allocation avoids wasting space due to internal fragmentation. Internal fragmentation refers to having unused portions of the 64 MB chunk. For example, if we allocate a 64 MB chunk and only fill up 20MB, the remaining space is unused.

One disadvantage of having a large chunk size is the handling of small files. Since a small file will have one or a few chunks, the ChunkServers storing those chunks can become hotspots if a lot of clients access the same file. To handle this scenario, GFS stores such files with a higher replication factor and also adds a random delay in the start times of the applications accessing these files.

# MASTER FUNCTIONS
## METADATA
The master stores three types of metadata:
* The file and chunk namespaces (i.e., directory hierarchy).
* The mapping from files to chunks.
* The locations of each chunk’s replicas.

### Storing metadata in memory
Since metadata is stored in memory, the master operates very quickly. Additionally, it is easy and efficient for the master to periodically scan through its entire state in the background. This periodic scanning is used to implement three functions:
* Chunk garbage collection
* Re-replication in the case of ChunkServer failures
* Chunk migration to balance load and disk-space usage across ChunkServers

As discussed above, one potential concern for this memory-only approach is that the number of chunks, and hence the capacity of the whole system, is limited by how much memory the master has. This is not a serious problem in practice. The master maintains less than 64 bytes of metadata for each 64 MB chunk. Most chunks are full because most files contain many chunks, only the last of which may be partially filled. Similarly, the file namespace data typically requires less than 64 bytes per file because the master stores file names compactly using prefix compression.

If the need for supporting an even larger file system arises, the cost of adding extra memory to the master is a small price to pay for the simplicity, reliability, performance, and flexibility gained by storing the metadata in memory.

### Chunk location
The master does not keep a persistent record of which ChunkServers have a replica of a given chunk; instead, the master asks each chunk server about its chunks at master startup, and whenever a ChunkServer joins the cluster. The master can keep itself up-to-date after that because it controls all chunk placements and monitors ChunkServer status with regular HeartBeat messages.

By having the ChunkServer as the ultimate source of truth of each chunk’s location, GFS eliminates the problem of keeping the master and ChunkServers in sync. It is not beneficial to maintain a consistent view of chunk locations on the master, because errors on a ChunkServer may cause chunks to vanish spontaneously (e.g., a disk may go bad and be disabled, or ChunkServer is renamed or failed, etc.) In a cluster with hundreds of servers, these events happen all too often.

### Operation log
* The master maintains an operation log that contains the namespace and file-to-chunk mappings and stores it on the local disk. Specifically, this log stores a historical record of all the metadata changes. Operation log is very important to GFS. It contains the persistent record of metadata and serves as a logical timeline that defines the order of concurrent operations.
* For fault tolerance and reliability, this operation log is replicated on multiple remote machines, and changes to the metadata are not made visible to clients until they have been persisted on all replicas. The master batches several log records together before flushing, thereby reducing the impact of flushing and replicating on overall system throughput.
* Upon restart, the master can restore its file-system state by replaying the operation log. This log must be kept small to minimize the startup time, and that is achieved by periodically checkpointing it.

### Checkpointing
Master’s state is periodically serialized to disk and then replicated, so that on recovery, a master may load the checkpoint into memory, replay any subsequent operations from the operation log, and be available again very quickly. To further speed up the recovery and improve availability, GFS stores the checkpoint in a compact B-tree like format that can be directly mapped into memory and used for namespace lookup without extra parsing.

The checkpoint process can take time, therefore, to avoid delaying incoming mutations, the master switches to a new log file and creates the new checkpoint in a separate thread. The new checkpoint includes all mutations before the switch.

## OPERATIONS
The master executes all namespace operations. Furthermore, it manages chunk replicas throughout the system. It is responsible for:
* Making replica placement decisions
* Creating new chunks and hence replicas
* Making sure that chunks are fully replicated according to the replication factor
* Balancing the load across all the ChunkServers
* Reclaim unused storage

### Namespace management and locking



