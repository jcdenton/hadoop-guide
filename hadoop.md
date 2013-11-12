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
 - ```FAIL``` DryadLinq - Microsoft Research research project (lol), based on Linq and Microsoft Dryad (*failed - Microsoft had finally chosen Apache Hadoop for their Azure platform*).
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

Like Hadoop Map/Reduce, HDFS follows a master/slave architecture. An HDFS installation consists of a single NameNode, a master server that manages the filesystem namespace and regulates access to files by clients. In addition, there are a number of Datanodes, one per node in the cluster, which manage storage attached to the nodes that they run on. The NameNode makes filesystem namespace operations like opening, closing, renaming etc. of files and directories available via an RPC interface. It also determines the mapping of blocks to DataNodes. The DataNodes are responsible for serving read and write requests from filesystem clients, they also perform block creation, deletion, and replication upon instruction from the NameNode.

## Data flow

Minimally, applications specify the input/output locations and supply map and reduce functions via implementations of appropriate interfaces and/or abstract classes. These, and other job parameters, comprise the job configuration. The Hadoop job client then submits the job (jar/executable etc.) and configuration to the JobTracker which then assumes the responsibility of distributing the software/configuration to the slaves, scheduling tasks and monitoring them, providing status and diagnostic information to the job-client.

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

[`RU` BigData Dive '13 - Обзор дистрибутивов Hadoop](http://www.slideshare.net/Lavrentieva/hadoop-hadoop)

 - Apache Hadoop
 - Cloudera CDH
 - Hortonworks HDP
 - MapR 
 - PivotalHD
 - Intel Hadoop Distribution


## Links & references

 - [Projects using Apache Hadoop](http://wiki.apache.org/hadoop/PoweredBy)
 - [Yahoo Hadoop blog](http://developer.yahoo.com/blogs/hadoop)
 - [Pythian BigData Videos](http://www.youtube.com/playlist?list=PLbJSpUhUkuUisc-13zbAEDcwdRw20wnGR)

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
