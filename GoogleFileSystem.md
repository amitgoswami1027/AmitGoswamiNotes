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




