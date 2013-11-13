# **H-stack Introduction Guide**

*Or just a ["let me google that for you"](http://lmgtfy.com/) article*

--------------------------------

# Table of Contents

- [MapReduce paradigm](#mapreduce-paradigm)
	- [Basics](#basics)
	- [MapReduce implementations](#mapreduce-implementations)
- [Apache Hadoop](#apache-hadoop)
	- [Overview](#overview)
	- [Architecture](#architecture)
	    - [Infrastructure](#infrastructure)
        - [Apache HDFS](#apache-hdfs)
        - [Data flow](#data-flow)
	- [API](#API)
	- [Distributions](#distributions)
	- [Links & references](#links-&-references)
- [Apache HBase](#apache-hbase)
	- [Inverted index](#inverted-index)
	- [Column-oriented DBMS](#column-oriented-dbms)
	- [API Bindings](#api-bindings)
	- [Links & references](#links-&-references-1)
- [Apache Hive](#apache-hive)
	- [Overview](#overview-1)
	- [Internal workflow](#internal-workflow)
	- [HiveQL](#hiveql)
		- [Definition & features](#definition-&-features)
		- [Syntax basics](#syntax-basics)
		- [HiveQL/SQL comparison](#hiveqlsql-comparison)
	- [Examples](#examples-1)
	- [Links & references](#links-&-references-2)
- [Apache Pig](#apache-pig)
	- [Overview](#overview-2)
	- [Execution modes](#execution-modes)
	- [PigLatin](#piglatin)
		- [Definition & features](#definition-&-features-1)
		- [Syntax basics](#syntax-basics-1)
		- [PigLatin/HiveQL comparison](#piglatinhiveql-comparison)
	- [Examples](#examples-2)
	- [Links & references](#links-&-references-3)
- [Other tools](#other-tools)
	- [Apache Sqoop](#apache-sqoop)
	- [Cascading](#cascading)
	- [Scalding](#scalding)

--------------------------------

# MapReduce paradigm

## Basics

[MapReduce](http://en.wikipedia.org/wiki/MapReduce) is a programming model for processing large data sets with a parallel, distributed algorithm on a cluster. 

The model is inspired by the map and reduce functions commonly used in functional programming, although their purpose in the MapReduce framework is not the same as their original forms. The core idea behind MapReduce is mapping your dataset into a collection of <key, value> pairs, and then reducing over all pairs with the same key. The overall concept is simple, but is actually quite expressive when you consider that:

 - almost all data can be mapped into <key, value> pairs somehow, and 
 - your keys and values may be of any type: strings, integers, dummy types... and, of course, <key, value> pairs themselves.

The canonical MapReduce use case is counting word frequencies in a large text, but some examples of what you can do in the MapReduce framework include:

 - distributed sort 
 - distributed search 
 - web-link graph traversal 
 - machine learning

> ![MapReduce WordCount example](https://dl.dropboxusercontent.com/u/6870375/word_count.png)

A MapReduce program is composed of a Map() procedure that performs filtering and sorting and a Reduce() procedure that performs a summary operation. The "MapReduce System" (also called "infrastructure", "framework") orchestrates by marshalling the distributed servers, running the various tasks in parallel, managing all communications and data transfers between the various parts of the system, providing for redundancy and fault tolerance, and overall management of the whole process. 

So, considering the diagram above, the framework takes all stages of processing except, actually, the "mapping" and "reducing" ones, i.e. it reads input from somewhere (stdin, database, etc.), splits it (optionally, transfers it to different nodes), preprocesses before the "reducing" phase and collects output to the one result.

## MapReduce implementations

 - Google proprietary MapReduce implementation in C++ (has Python and Java bindings, available at GAE).
 - Apache Hadoop - open-source software framework written in Java, has numerous distributions and is part of many integrated solutions.
 - Disco - lightweight open-source Erlang framework written by Nokia (includes super simple Python API).
 - CouchDB internal MapReduce implementation.
 - MongoDB internal MapReduce implementation.
 - Phoenix - a MapReduce framework for shared-memory CMPs and SMPs, written in C.
 - Qt Concurrent - includes a simplified MapReduce and FilterReduce implementation for shared-memory (non-distributed) systems.
 - Greenplum MapReduce (although they have a Hadoop distribution - Greenplum HD).
 - Skynet - open-source Ruby implementation.
 - Qizmt - C# implementation by MySpace.
 - Cell Broadband Engine implementation in C.
 - `FAIL` DryadLinq - Microsoft Research research project (lol), based on Linq and Microsoft Dryad (*failed - Microsoft had finally chosen Apache Hadoop for their Azure platform*).
 - And a legion of Hadoop-based implementations.

--------------------------------

# Apache Hadoop

##  Overview

The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.

The project includes these modules:

 - Hadoop Common: The common utilities that support the other Hadoop
   modules. 
 - Hadoop Distributed File System (HDFS): A distributed file system that provides high-throughput access to application data.
 - Hadoop YARN: A framework for job scheduling and cluster resource management. 
 - Hadoop MapReduce: A YARN-based system for parallel processing of large data sets.

Other Hadoop-related projects at Apache include:

 - Ambari: A web-based tool for provisioning, managing, and monitoring Apache Hadoop clusters which includes support for Hadoop HDFS, Hadoop MapReduce, Hive, HCatalog, HBase, ZooKeeper, Oozie, Pig and Sqoop. Ambari also provides a dashboard for viewing cluster health such as heatmaps and ability to view MapReduce, Pig and Hive applications visually alongwith features to diagnose their performance characteristics in a user-friendly manner. 
 - Avro: A data serialization system.
 - Cassandra: A scalable multi-master database with no single points of failure.
 - Chukwa: A data collection system for managing large distributed systems.
 - HBase: A scalable, distributed database that supports structured data storage for large tables.
 - Hive: A data warehouse infrastructure that provides data summarization and ad hoc querying.
 - Mahout: A Scalable machine learning and data mining library.
 - Pig: A high-level data-flow language and execution framework for parallel computation.
 - ZooKeeper: A high-performance coordination service for distributed applications.

## Architecture

> ![Apache Hadoop topology architecture](http://upload.wikimedia.org/wikipedia/en/2/2b/Hadoop_1.png)

### Infrastructure

The Hadoop Map/Reduce framework has a master/slave architecture and because of it's batch processing nature, Hadoop operates with so called *jobs*. It has a single master server or JobTracker and several slave servers or TaskTrackers, one per node in the cluster. The JobTracker is the point of interaction between users and the framework. Users submit map/reduce jobs to the JobTracker, which puts them in a queue of pending jobs and executes them on a first-come/first-served basis. The JobTracker manages the assignment of map and reduce tasks to the TaskTrackers and re-executes the failed tasks. The TaskTrackers execute tasks upon instruction from the JobTracker and also handle data motion between the map and reduce phases.

Typically the compute nodes and the storage nodes are the same, that is, the MapReduce framework and the Hadoop Distributed File System are running on the same set of nodes. This configuration allows the framework to effectively schedule tasks on the nodes where data is already present, resulting in very high aggregate bandwidth across the cluster.

### Apache HDFS

A distributed file system is designed to hold a large amount of data and provide access to this data to many clients distributed across a network. There are a number of distributed file systems that solve this problem in different ways.
NFS, the Network File System, is the most ubiquitous distributed file system. It is one of the oldest still in use. While its design is straightforward, it is also very constrained. NFS provides remote access to a single logical volume stored on a single machine. An NFS server makes a portion of its local file system visible to external clients. The clients can then mount this remote file system directly into their own Linux file system, and interact with it as though it were part of the local drive.

One of the primary advantages of this model is its transparency. Clients do not need to be particularly aware that they are working on files stored remotely. The existing standard library methods like `open()`, `close()`, `fread()`, etc. will work on files hosted over NFS.

But as a distributed file system, it is limited in its power. The files in an NFS volume all reside on a single machine. This means that it will only store as much information as can be stored in one machine, and does not provide any reliability guarantees if that machine goes down (e.g., by replicating the files to other servers). Finally, as all the data is stored on a single machine, all the clients must go to this machine to retrieve their data. This can overload the server if a large number of clients must be handled. Clients must also always copy the data to their local machines before they can operate on it.

Like Hadoop Map/Reduce, HDFS follows a master/slave architecture. An HDFS installation consists of a single NameNode, a master server that manages the filesystem namespace and regulates access to files by clients. In addition, there are a number of Datanodes, one per node in the cluster, which manage storage attached to the nodes that they run on. The NameNode makes filesystem namespace operations like opening, closing, renaming etc. of files and directories available via an RPC interface. It also determines the mapping of blocks to DataNodes. The DataNodes are responsible for serving read and write requests from filesystem clients, they also perform block creation, deletion, and replication upon instruction from the NameNode.

 - HDFS is designed to be robust to a number of the problems that other DFS's such as NFS are vulnerable to. In particular:
 - HDFS is designed to store a very large amount of information (terabytes or petabytes). This requires spreading the data across a large number of machines. It also supports much larger file sizes than NFS.
 - HDFS should store data reliably. If individual machines in the cluster malfunction, data should still be available.
 - HDFS should provide fast, scalable access to this information. It should be possible to serve a larger number of clients by simply adding more machines to the cluster.
 - HDFS should integrate well with Hadoop MapReduce, allowing data to be read and computed upon locally when possible.

But while HDFS is very scalable, its high performance design also restricts it to a particular class of applications; it is not as general-purpose as NFS. There are a large number of additional decisions and trade-offs that were made with HDFS. In particular:

 - Applications that use HDFS are assumed to perform long sequential streaming reads from files. HDFS is optimized to provide streaming read performance; this comes at the expense of random seek times to arbitrary positions in files.
 - Data will be written to the HDFS once and then read several times; updates to files after they have already been closed are not supported. (An extension to Hadoop will provide support for appending new data to the ends of files; it is scheduled to be included in Hadoop 0.19 but is not available yet.)
 - Due to the large size of files, and the sequential nature of reads, the system does not provide a mechanism for local caching of data. The overhead of caching is great enough that data should simply be re-read from HDFS source.
 - Individual machines are assumed to fail on a frequent basis, both permanently and intermittently. The cluster must be able to withstand the complete failure of several machines, possibly many happening at the same time (e.g., if a rack fails all together). While performance may degrade proportional to the number of machines lost, the system as a whole should not become overly slow, nor should information be lost. Data replication strategies combat this problem.
The design of HDFS is based on the design of GFS, the Google File System. Its design was described in a paper published by Google.

DataNodes holding blocks of multiple files with a replication factor of 2. The NameNode maps the filenames onto the block IDs:
![DataNodes holding blocks of multiple files with a replication factor of 2. The NameNode maps the filenames onto the block IDs](http://farm3.static.flickr.com/2050/3529146393_5c2e2c8065_o.png)


## Data flow

Minimally, applications specify the input/output locations and supply map and reduce functions via implementations of appropriate interfaces and/or abstract classes. These, and other job parameters, comprise the job configuration. The Hadoop job client then submits the job (jar/executable etc.) and configuration to the JobTracker which then assumes the responsibility of distributing the software/configuration to the slaves, scheduling tasks and monitoring them, providing status and diagnostic information to the job-client.

Common data flow is showed on the following diagram:

![Common data flow diagram](http://farm3.static.flickr.com/2275/3529146683_c8247ff6db_o.png)

## API

 - General workflow
 - JobConf
 - InputFormat
 - Mapper
 - Combiner
 - Partitioner
 - Reducer
 - OutputFormat

## Distributions

The most popular Hadoop distributions include:

 - Apache Hadoop
 - Cloudera CDH
 - Hortonworks HDP
 - MapR 
 - PivotalHD
 - Intel Hadoop Distribution

> `RU` [BigData Dive '13 - Обзор дистрибутивов Hadoop](http://www.slideshare.net/Lavrentieva/hadoop-hadoop)
> 
> [![BigData Dive '13 - Обзор дистрибутивов Hadoop](http://img.youtube.com/vi/Is0N6TSptNY/0.jpg)](http://www.youtube.com/watch?v=Is0N6TSptNY)

## Links & references

 - [Yahoo Hadoop blog](http://developer.yahoo.com/blogs/hadoop) *(very usefull and well-structured)*
 - [Pythian BigData Videos](http://www.youtube.com/playlist?list=PLbJSpUhUkuUisc-13zbAEDcwdRw20wnGR) *(lots of short overview movies on different BigData technologies incl. Apache Hadoop)*
 - [Hadoop best practices and design patterns](http://developer.yahoo.com/blogs/hadoop/apache-hadoop-best-practices-anti-patterns-465.html)
 - [Projects using Apache Hadoop](http://wiki.apache.org/hadoop/PoweredBy)

--------------------------------

# Apache HBase

## Inverted index
## Column-oriented DBMS
## API Bindings
## Links & references

--------------------------------
 
# Apache Hive

## Overview
## Internal workflow
## HiveQL
### Definition & features
### Syntax basics
### HiveQL/SQL comparison
## Examples
## Links & references

--------------------------------

# Apache Pig

## Overview
## Execution modes
## PigLatin
### Definition & features
### Syntax basics
### PigLatin/HiveQL comparison
## Examples
## Links & references

--------------------------------

# Other tools

## Apache Sqoop
## Cascading
## Scalding
