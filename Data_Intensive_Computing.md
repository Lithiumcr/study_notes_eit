---
Author: August
Title: Advanced Internetworking（Amir H. Payberah, et al.）
Office: Room 3425, Floor 4, Electrum 229, Kistagången 16, 16440 Kista, Sweden; Q&A Also https://docs.google.com/document/d/1Og6V4okXNLmsCwmRRAmKEMuQuTUrZBwmRkQfwYNwVoo/edit#heading=h.844hsjjugkn4
Tel: +46-(0)72-55 44 011
Email: payberah@kth.se
Web: https://payberah.github.io/
Lecture Nodes (Very Important!): https://id2221kth.github.io/
Notebook: https://id2221kth.github.io/material/
Score: 

---

# Advanced Internetworking

[TOC]

## 2023-08-29

### introduction

Syllabus

tasks

group

grade

### Cloud Computing

#### The NIST definition

* Five characteristics
  * On-demand Self-Service: A consumer can independently provision computing capabilities without human interaction with the service provider.
  * Ubiquitous Network Access
  * Resource Pooling: Provider’s computing resources are pooled to serve consumers
  * Rapid Elasticity: Capabilities can be rapidly and elastically provisioned, in some cases automatically.
  * Measured Service: Resource usage can be monitored, controlled, and reported providing transparency for both the provider and consumer.
* Three service models
  * Infrastructure as a Service (IaaS): similar to building a new house. 
  * Platform as a Service (PaaS): similar to buying an empty house. 
  * Software as a Service (SaaS): similar to living in a hotel.
* Four deployment models
* ![img](https://s7280.pcdn.co/wp-content/uploads/2017/09/saas-vs-paas-vs-iaas.png)

### Big Data

buzzwords: data characterized by 4 key attributes: volume, variety, velocity and value.

Traditional platforms fail to show the expected performance.

3-level model

* resource management
* data storage
* data processing

## 2023-09-01 file system

### definition

Controls how data is stored in and retrieved from storage device.

Distributed file systems: manage the storage across a network of machines.

### Google File System (GFS)

#### Motivation and Assumptions

* Huge files (multi-GB)
* Most files are modified by appending to the end
  * Random writes (and overwrites) are practically non-existent
* Optimise for streaming access (Write once, read many.)
* Node failures happen frequently

#### Chunk

single unit of storage. 

* **Immutable** and globally unique chunk handle
* **Transparent** to user
* Each chunk is stored as a **plain Linux file**

#### Architecture

* master
  * Responsible for all system-wide activities 
    * Maintains all file system metadata: 
    * Namespaces, ACLs, mappings from files to chunks, and current locations of chunks
    * All kept in memory, namespaces and file-to-chunk mappings are also stored persistently in operation log 
  * Periodically communicates with each chunkserver: 
    * Determines chunk locations
    * Assesses state of the overall system
* chunkserver
  * Manages chunks 
  * Tells master what chunks it has 
  * Stores chunks as files 
  * Maintains data consistency of chunks
* client
  * Issues control requests to master server. 
  * Issues data requests directly to chunkservers. 
  * Caches metadata. 
  * Does not cache data.

#### Data Flow and Control Flow 

* Data flow is decoupled from control flow 
* Clients interact with the master for metadata operations (control flow) 
* Clients interact directly with chunkservers for all files operations (data flow)

#### Why Large Chunks?

* 64MB or 128MB (much larger than most file systems)
* Advantages 
  * Reduces the size of the metadata stored in master
  * Reduces clients’ need to interact with master 
* Disadvantages: Wasted space due to internal fragmentation

### System interactions

System interface

* Not POSIX-compliant, but supports typical file system operations 
  * create, delete, open, close, read, and write
* snapshot: creates a copy of a file or a directory tree at low cost 
* append: allow multiple clients to append data to the same file concurrently

#### Read Operation

1. Application originates the read request. 
2. GFS client translates request and sends it to the master. 
3. The master responds with chunk handle and replica locations.
4. The client picks a location and sends the request. 
5. The chunkserver sends requested data to the client. 
6. The client forwards the data to the application.

---

1. 应用程序发起读请求。
2. GFS客户端翻译请求并发送给master。
3. 主服务器以块句柄和副本位置进行响应。
4. 客户端选择一个位置并发送请求。
5. chunkserver将请求的数据发送给客户端。
6. 客户端将数据转发给应用程序。

#### Update Order

* Update (mutation): an operation that changes the content or metadata of a chunk. 
* For consistency, updates to each chunk must be ordered in the same way at the different chunk replicas. 
* Consistency means that replicas will end up with the same version of the data and not diverge.
* For this reason, for each chunk, one replica is designated as the primary.
* The other replicas are designated as secondaries.
* Primary defines the update order.
* All secondaries follow this order.

---

* 更新（变易）：更改块的内容或元数据的操作。
* 为了一致性，每个块的更新必须在不同的块副本上以相同的方式排序。
* 一致性意味着副本最终将得到相同版本的数据并且不会出现分歧。
* 因此，对于每个块，都会指定一个副本作为主副本。
* 其他副本被指定为次级。
* 一级节点定义更新顺序。
* 所有次级都遵循此顺序。

#### Primary Leases

* For correctness there needs to be one single primary for each chunk. 
* At any time, at most one server is primary for each chunk. 
* Master selects a chunkserver and grants it lease for a chunk.
* The chunkserver holds the lease for a period T after it gets it, and behaves as primary during this period. 
* If master does not hear from primary chunkserver for a period, it gives the lease to someone else.

---

* 为了正确性，每个块需要有一个一级节点。
* 在任何时候，对于每个块最多有一台服务器是一级。
* Master 选择一个 chunkserver 并授予它租用lease一个 chunk。
* chunkserver在获得租约后将其保留一段时间T，在此期间充当一级。
* master 在一段时间内没有收到一级 chunkserver 的报文，将把租约交给其他人。

#### Write Operation

1. Application originates the request. 
2. The GFS client translates request and sends it to the master. 
3. The master responds with chunk handle and replica locations.
4. The client pushes write data to all locations. Data is stored in chunkserver’s internal buffers.
5. The client sends write command to the primary.
6. The primary determines serial order for data instances in its buffer and writes the instances in that order to the chunk.
7. The primary sends the serial order to the secondaries and tells them to perform the write.

---
1. 应用程序发起请求。
2. GFS 客户端翻译请求并将其发送给主服务器。
3. 主服务器以块句柄和副本位置进行响应。
4. 客户端将写数据推送到所有位置。 数据存储在 chunkserver 的内部缓冲区中。
5. 客户端向一级节点发送写命令。
6. 一级节点确定其缓冲区中数据实例的串行顺序，并按该顺序将实例写入块。
7. 一级节点将串行命令发送到次级节点并告诉它们执行写入。

#### Write Consistency

* Primary enforces one update order across all replicas for concurrent writes. 
* It also waits until a write finishes at the other replicas before it replies. 
* Therefore: 
  * We will have identical replicas.
  * But, file region may end up containing mingled fragments from different clients: e.g., writes to different chunks may be ordered differently by their different primary chunkservers
  * Thus, writes are consistent but undefined state in GFS

---

* 一级节点在所有副本中强制执行一个更新顺序以进行并发写入。
* 它还会等待其他副本的写入完成后再进行回复。
* 所以：
   * 我们将有相同的复制品。
   * 但是，文件区域最终可能包含来自不同客户端的混合片段：例如，对不同块的写入可能会由不同的一级服务器以不同的方式排序
   * 因此，GFS 中的写入是一致但未定义的状态

#### Append Operation

1. Application originates record append request.
2. The client translates request and sends it to the master.
3. The master responds with chunk handle and replica locations.
4. The client pushes write data to all locations.
5. The primary checks if record fits in specified chunk.
6. If record does not fit, then the primary:
* Pads the chunk,
* Tells secondaries to do the same,
* And informs the client.
* The client then retries the append with the next chunk.
7. If record fits, then the primary:
* Appends the record,
* Tells secondaries to do the same,
* Receives responses from secondaries,
* And sends final response to the client

---

1. 应用程序发起记录追加请求。
2. 客户端翻译请求并将其发送给主站。
3. 主服务器以块句柄和副本位置进行响应。
4. 客户端将写数据推送到所有位置。
5. 一级检查记录是否适合指定的块。
6. 如果记录不适合，则主要：
* 填充块，
* 告诉次级也这样做，
* 并通知客户。
* 然后客户端重试附加下一个块。
7. 如果记录符合，则主要：
* 附加记录，
* 告诉次级也这样做，
* 接收来自次级的响应，
* 并将最终响应发送给客户端

#### Delete Operation

* Metadata operation.
* Renames file to special name.
* After certain time, deletes the actual chunks. 
* Supports undelete for limited time. 
* Actual lazy garbage collection

### The Master Operations

#### A Single Master

#### The Master Operations

#### Namespace Management and Locking

### Fault Tolerance

### GFS and HDFS

### Summary

## 2023-09-04 NoSQL Databases
Database and Database Management System

Database: an organized collection of data.

Database Management System (DBMS): a software to capture and analyze data.

### Relational SQL DB 
#### ACID

1.Atomicity
  All included statements in a transaction are either executed or the whole transaction is aborted without affecting the database.

2. Consistency
    A database is in a consistent state before and after a transaction.
3. Isolation
    Transactions can not see uncommitted changes in the database.
4. Durability
    Changes are written to a disk before a database commits a transaction so that committed data cannot be lost through a power failure.

SQL Databases Challenges: Web-based applications caused spikes.

* Internet-scale data size
* High read-write rates
* Frequent schema changes

**RDBMS were not designed to be distributed.**

### NoSQL

* Avoids:
  * Overhead of ACID properties
  * Complexity of SQL query
* Provides:
  * Scalablity
  * Easy and frequent changes to DB
  * Large data volumes

#### Availability

Replicating data to improve the availability of data.

Data replication: Storing data in more than one site or node

#### Consistency

Strong consistency

After an update completes, any subsequent access will return the updated value

Eventual consistency
* Does not guarantee that subsequent accesses will return the updated value.
* Inconsistency window.
* If no new updates are made to the object, eventually all accesses will return the last updated value.

The large-scale applications have to be reliable: consistency, availability, partition
tolerance

Achieving ACID properties on large-scale applications is challenging.

#### CAP theorem

You can choose only two among CAP!

Continue the operation in the presence of network partitions.

#### NoSQL Data Models

##### Key-Value Data Model

Collection of KV Pairs

Ordered Key-Value: processing over key ranges.

Dynamo, Scalaris, Voldemort, Riak, ...

##### Column-Oriented Data Model

Similar to a key/value store, but the value can have multiple attributes

Store and process data by column instead of row.

BigTable, Hbase, Cassandra, ...

##### Document Data Model

Similar to a column-oriented store, but values can have complex documents.

Flexible schema (XML, YAML, JSON, and BSON).

CouchDB, MongoDB, ...

##### Graph Data Model

graph structures with nodes, edges, and properties

Neo4J, InfoGrid, ...

### BigTable

* Lots of (semi-)structured data at Google. 
  * URLs, per-user data, geographical locations, ... 
* Distributed multi-level map 
* **CP**: strong consistency and partition tolerance

#### Data Model

* Table: Distributed multi-dimensional sparse map
* Rows
  * read or write in a row is atomic.
  sorted in lexicographical order.
* Column
  * basic unit of data access.
  * Column families: group of (the same type) column keys
  * Column key naming: ``family:qualifier``
* Timestamp
* Tablet: contiguous ranges of rows stored together
  * split by the system when they become too large.
  * Each tablet is served by exactly one tablet server.

#### System Architecture

* Master
* Tablet Server
* Client Lib.
* Building Blocks
  * GFS
  * Chubby
  * SSTable

#### Tablet Serving

* Master Startup
* Tablet Assignment
* Finding a Tablet
  * Three-level hierarchy: Chubby, Root tablet, METADATA tablets
* Tablet Serving
  * Updates committed to a commit log. 
  * Recently committed updates are stored in memory - memtable 
  * Older updates are stored in a sequence of SSTables

Strong consistency: 

* **Only one tablet server is responsible for a given piece of data.** 
* **Replication is handled on the GFS layer.** 

Trade-off with availability: If a tablet server fails, its portion of data is **temporarily unavailable** until a new server is assigned.

#### BigTable vs. HBase



### Cassandra

column-oriented, for FB, **AP**

HASH

### Neo4j

* graph database 
* The relationships between data is equally important as the data itself 
* Cypher: a declarative query language similar to SQL, but optimized for graphs 
* **CA**: strong consistency and availability

#### Data Model

* Node (Vertex)
  * main data element
  * A waypoint along a traversal route
* Relationship (Edge)
  * May contain Direction, Metadata, e.g., weight or relationship type
* Label
  * Define node category, 0~Multiple
* Properties
  * Enrich a node or relationship

#### Cypher

Declarative query language

(): Nodes | []: Relationships | {}: Properties

### Summary

## 2023-09-06 Scala

scalable language. A blend of object-oriented and functional programming. Runs on the Java Virtual Machine.

### Scala basics

Val: immutable

Var: mutable

Always use immutable values by default

### Functions

Nested Functions

Anonymous Functions

Higher-Order Functions

Call-by-Name: the value of the parameter is not determined until it is called within the function.

### Collections

Arrays 定长数组

Lists 不可变列表

Sets 去重集

Maps 键值对词典

Functional Combinators

map 分量变换list, foreach分量提取list, filter滤除, zip 合并list, partition, find按条件取首个, drop 按长截取 and dropWhile 条件截取, foldLeft, foldRight, flatten, flatMap

### Classes and objects

tuples

option

either

**Everything in Scala is an Object**

Classes and Objects

Inheritance and Overloading Methods

Singleton

Abstract Class

Traits

Case Classes and Pattern Matching

### Simple Build Tool - SBT

## 2023-09-11 MapReduce 映射规约

Scale up 单元强化

Scale out 扩展

A shared nothing architecture for processing large data sets with a parallel/distributed algorithm on clusters of commodity hardware.

MapReduce takes care of parallelization, fault tolerance, and data distribution; Hide system-level details from programmers.

### Programming Model

Data-Parallel Processing

**Map -> Shuffle && Sort -> Reduce** 

example: word count of a file distributed stored

Parallelize the data and process;

* Each Map task (typically) operates on a single HDFS block.
* Map tasks (usually) run on the node where the block is stored.
* Each Map task generates a set of intermediate key/value pairs.
* Sorts and consolidates intermediate data from all mappers, after all Map tasks are complete and before Reduce tasks start;
* Each Reduce task operates on all intermediate values associated with the same intermediate key. 
* Produces the final output.

Data Flow

#### Mapper

Input: (key, value) pairs 

Output: a list of (key, value) pairs

The Mapper may use or completely ignore the input key.

A standard pattern is to read one line of a file at a time. Key: the byte offset; Value: the content of the line. 

After the Map phase, all intermediate (key, value) pairs are grouped by the intermediate keys, **(key, list of values)** passed to a Reducer

#### Reducer

Input: (key, list of values) pairs 

Output: zero or more final (key, value) pairs



#### MapReduce Algorithm Design

* Local aggregation
  * In-Map Combiner
* Joining, relational constructs you use to combine relations together.
  * Reduce-side
    * Repartition join, two or more large datasets together
  * Map-side
    * Replication join, When one of the datasets is small enough to cache
* Sorting, give output in total sort order
  * For multiple Reducers we need to choose a partitioning function

### Implementation

**Hadoop**, Hadoop Distributed File System, HDFS

MapReduce Execution

1. The user program divides the input files into M splits, size of a HDFS block (64 MB), Converts them to **key/value pairs**; starts up many copies of the program on a cluster.
2. The **master** picks idle workers and assigns each one a map task or a reduce task.
3. A map worker reads the contents of the corresponding input splits. key/value pairs produced by the **user defined map function** are buffered in memory.
4. The buffered pairs are periodically written to local disk, partitioned into R regions (hash(key) mod R); Locations of the buffered pairs on the local disk are passed back to the master to forward these locations to the reduce workers.
5. A **reduce worker remotely reads the buffered data** from the local disks of the map workers, then sorts it by the intermediate keys.
6. reduce worker iterates over the intermediate data: For each unique intermediate key, it passes the key and the corresponding set of intermediate values to the **user defined reduce function**. The output of the reduce function is appended to a final output file.
7. master wakes up the user program.

---

1. 用户程序将输入文件分为M个分片，大小为HDFS块（64 MB），将它们转换为**键/值对**； 在集群上启动该程序的多个副本。
2. **master**挑选空闲的worker并给每个分配一个map任务或一个reduce任务。
3. 地图工作者读取相应输入分割的内容。 由**用户定义的映射函数**生成的键/值对缓冲在内存中。
4. 缓冲的对定期写入本地磁盘，划分为 R 个区域（hash(key) mod R）； 本地磁盘上缓冲对的位置被传回主节点，以将这些位置转发给reduce工作节点。
5. **Reduce Worker 从 Map Worker 的本地磁盘远程读取缓冲数据**，然后根据中间键对其进行排序。
6. reduceworker迭代中间数据：对于每个唯一的中间键，它将键和相应的中间值集传递给**用户定义的reduce函数**。 reduce 函数的输出被附加到最终输出文件中。
7. master唤醒用户程序。

#### Fault Tolerance

Worker

* Detect failure via **periodic heartbeats**.

* Re-execute in-progress map and reduce tasks.
* Re-execute completed map tasks: their output is stored on the local disk of the failed machine and is therefore inaccessible.
* Completed reduce tasks do not need to be re-executed since their output is stored in a global filesystem.

Master

State is periodically checkpointed: a new copy of master starts from the last checkpoint state.

## 2023-09-12 Spark

> H. Karau et al., High Performance Spark, O’Reilly Media, 2017  

Spark: Another way to batch data

Motivation: **Acyclic data flow** from stable storage to stable storage during MapReduce

### Spark Applications Architecture 

**1 Driver + some executors** based on the Cluster manager process

Driver Process, SparkSession, main()

* Maintaining information about the Spark application
* Responding to a user’s program or input
* Analyzing, distributing, and scheduling work across the executors

Executor Processes

* Executing code assigned to it by the driver

* Reporting the state of the computation on that executor back to the driver 

SparkSession

A one-to-one correspondence between a SparkSession and a Spark application. Available in console shell as `spark `

SparkContext  

The entry point for low-level AP* functionality, access it through the SparkSession

Available in console shell as `sc`  

* Prior to Spark 2.0.0, a the spark driver program uses SparkContext to connect to
the cluster.
* In order to use APIs of SQL, Hive and streaming, **separate** SparkContexts should
to be created.
* SparkSession provides access to all the spark functionalities that SparkContext
does, e.g., SQL, Hive and streaming.
* **SparkSession internally has a SparkContext for actual computation**  

### Programming Model

**directed acyclic graphs (DAG) 有向无环图** data flow

A data flow is composed of any number of **data sources, operators, and data sinks** by connecting their inputs and outputs.

Parallelizable operators

**Resilient Distributed Datasets (RDD) 弹性分布式数据集**

* A distributed memory abstraction
* Immutable collections of objects spread across a cluster, like LinkedList
* An RDD is divided into a number of **partitions**, which are **atomic** pieces of information.
* Partitions of 1 RDD can be stored on different nodes of a cluster
* RDDs were the primary AP* in the Spark 1.x series.
* Virtually all Spark code you run, compiles down to an RDD

---

* 分布式内存抽象模型
* 分布在集群中的不可变对象集，如同 LinkedList
* RDD可被分为许多**分区**，**原子**的。
* 1个RDD的分区物理上可以存储在集群的不同节点上
* RDD 是 Spark 1.x 系列中的主要 API。
* 实际上，您运行的所有 Spark 代码都会编译为 RDD

Two types of RDDs: Generic RDD, Key-value RDD

KV RDD have special operations, such as aggregation, and a concept of
custom partitioning by key

Creating RDDs

* Use the parallelize method on a **SparkContext**.
* This turns a **single node collection** into a **parallel** collection.
* You can also explicitly state the number of partitions.
* In the console shell, you can either use `sc` or `spark.sparkContext`

Create RDD from an external storage. E.g., local file system, HDFS, Cassandra, HBase, Amazon S3, etc. Text file RDDs can be created using `textFile` method.

**RDD Operations: Transformation & Action**

#### Transformations  

Create a new RDD from an existing one

transformations are lazy:

* Not compute their results right away.
* Remember the transformations applied to the base dataset.
* They are only computed when an action requires a result to be returned to the driver program 

Lineage: transformations used to build an RDD. RDDs are stored as a chain of objects capturing the lineage of each RDD

**Generic RDD Transformations**

1. `filter` returns the RDD records that match some predicate function.
2. `distinct` removes duplicates from the RDD
3. `map` and `flatMap` apply a given function on each RDD record independently  
4. `sortBy` sorts an RDD records

**Key-Value RDD Transformations**

To make a key-value RDD:

1. `map` over your current RDD to a basic key-value structure.
2. Use the `zip` to zip together two RDD
3. `keys` and `values` extract keys and values, respectively.
4. `mapValues` maps over values

**Aggregation**

`groupByKey` or `reduceByKey`

**Join**

...

#### Actions

Transformations allow us to build up our logical transformation plan.

**action to trigger the computation.** (Instructs Spark to compute a result from a series of transformations.)

3 kinds of actions:

* Actions to view data in the console
* Actions to collect data to native objects in the respective language
* Actions to write to output data sources  

#### Example: Spark Word-Count

```scala
val textFile = sc.textFile("hdfs://...")
val words = textFile.flatMap(line => line.split(" "))
val ones = words.map(word => (word, 1))
val counts = ones.reduceByKey(_ + _)
counts.saveAsTextFile("hdfs://...")
```

#### Cache and Checkpoints

When you cache an RDD, each node stores any partitions of it that it computes in **memory**. An RDD that is **not cached is re-evaluated each time** an action is invoked on that RDD.

A node **reuses** the cached RDD in other actions on that dataset.

2 functions for caching an RDD:

* `cache` caches the RDD into memory
* `persist(level)` can cache in memory, on disk, or off-heap memory

`checkpoint` saves an RDD to disk.

Checkpointed data is **not removed** after `SparkContext` is destroyed. When we reference a checkpointed RDD, it will derive from the checkpoint instead of the source data.  

#### Execution Engine

**DAG** representing the computations done on the RDD is called **lineage graph 沿袭图**.

**RDD Dependencies** encode when data must move across network  

Narrow transformations (dependencies)  vs. Wide transformations (dependencies)  

Narrow: -> 1 partition, pipelining

Wide: -> many partitions, shuffle, e.g. groupBy, join with inputs not co-partitioned

**Lineages and Fault Tolerance**

**After Failure: Recompute only the lost partitions of an RDD**

#### Job, Stage, Task

the highest element of Spark’s execution hierarchy. Each Spark job corresponds to one action called by the driver program. 

Each **job** breaks down into a series of **stages**. 

**Stages** in Spark represent groups of **tasks** that can be executed together. 

**Task** is the smallest execution unit, representing one local computation.

All of the tasks in one stage execute the same code on a different piece of the data. (SIMD)

### Spark SQL - to structure data

RDDs don’t know anything about the **schema** of the data it’s dealing with

#### DataFrame, Schema, Column, Row

#### DataFrame Transformations

#### DataFrame Actions  

Aggregation, Join, SQL

DataSet, Structured Data Execution

#### Summary

### Lab intro





## 2023-09-18 Structured data processing

Structured data

distributed table-like collections with well-defined rows and columns.

They represent **immutable lazily** evaluated plans

### Spark DataFrame

Schema

Column

Row, record of data, stored distributed, does not have schema

Creating a DataFrame: 

from RDD, `toDF`

from raw data sources

#### DF Transformations

Add, remove, transform row <-> column, change order

aggregate 

group by

windowing

...

SQL in sparkSession

### Spark DataSet

more complex and strongly typed transformations



## 2023-09-20 Data stream processing

### Definition

the act of continuously incorporating new data to compute a result  

Database Management Systems (DBMS): data-at-rest analytics. (Store and index data before processing it; Process data only when explicitly asked by the users.)

**Stream Processing Systems (SPS): data-in-motion analytics**, Processing information as it flows, without storing them persistently

* Data stream is unbound data, which is broken into a sequence of individual tuples.

* A data tuple is the atomic data item in a data stream.
* Can be structured, semi-structured, and unstructured.

Micro-batch systems

Continuous processing-based systems

### Event and Processing Time

**Window**: a buffer associated with an input port to retain previously received tuples

**windowing management policies**

* Count-based policy: the maximum number of tuples a window buffer can hold
* Time-based policy: based on processing or event time period  

**types of windows**: tumbling & sliding

Tumbling: supports batch operations. When the buffer fills up, all the tuples are evicted  

Sliding: supports incremental operations. When the buffer fills up, older tuples are evicted

*def.* Event time: the time at which **events actually occurred**. Timestamps inserted into each record at the source.

Prcosseing time: the time when the record is **received at the streaming application**.

Skew

Triggering determines when in processing time the results of groupings are emitted
as panes; Windowing determines where in event time data are grouped together for processing. Time-based or data-driven triggers.

#### Time-based Triggering (Processing Time) 

The system buffers up incoming data into windows until some amount of processing
time has passed.

Reflect the times at which events actually happened while Handling out-of-order events  

* Watermarking helps a stream processing system to deal with lateness.
* Watermarks flow as part of the data stream and carry a timestamp t.
* A watermark is a threshold to specify how long the system waits for late events. 
* Streaming systems uses watermarks to measure progress in event time.
* A **W(t) declares that event time has reached time t in that stream**
  * There **should be no more elements from the stream with timestamp t' ≤ t**.
* It is possible that certain elements will violate the watermark condition.
  * After the W(t) has occurred, more elements with timestamp t' ≤ t will occur.
* If an arriving event lies within the watermark, it gets used to update a query.
* Streaming programs may explicitly expect some **late elements**  

#### Windowing and Triggering - Example

**Trigger at period (time-based triggers) / Trigger at count (data-driven triggers)** 

Fixed window, trigger at **period** (micro-batch) / Fixed window, trigger at **watermark** (streaming)

### Data Stream Storage

Messaging systems: notify consumers about new events

#### Direct messaging

* Necessary in **latency critical** applications (e.g., remote surgery).
* A producer sends a message containing the event, which is pushed to consumers. 推流
* Both consumers and producers have to be online at the same time.

#### Message brokers  

* message broker **decouples** the producer-consumer interaction.
* It runs as a **server**, with producers and consumers connecting to it as clients.
* Producers write messages to the **broker**, and consumers receive them by reading them from the broker.  
* Consumers are generally asynchronous.

#### Partitioned logs

* In typical message brokers, once a message is consumed, it is deleted.
* Log-based message brokers durably store all events in a sequential log.
* A log is an append-only sequence of records on disk.
* A producer sends a message by appending it to the end of the log.
* A consumer receives messages by reading the log sequentially.

---

* 在典型的报文代理中，一旦报文被消费就会被删除。
* 基于日志的报文代理将所有事件持久存储在顺序日志中。
* 日志是磁盘上仅可附加写入的记录序列。
* 生产者通过将报文附加到日志末尾来发送报文。
* 消费者通过顺序读取日志来接收报文。

#### Kafka - A Log-Based Message Broker

distributed, topic oriented, partitioned, replicated commit log service. 

##### Logs, Topics and Partition

* Kafka is about logs.
* Topics are queues: a stream of messages of a particular type
* Each message is assigned a sequential id called an offset
* Topics are logical collections of partitions (the physical files).
  * Ordered
  * Append only
  * Immutable
* Ordering is only guaranteed within a partition for a topic.
* Messages sent by a producer to a particular topic partition will be appended in the order they are sent.
* A consumer instance sees messages in the order they are stored in the log.
* Partitions of a topic are replicated: fault-tolerance
* A broker contains some of the partitions for a topic.
* One broker is the leader of a partition: all writes and reads must go to the leader

---

* 卡夫卡是围绕日志的。
* Topic 主题是队列：特定类型的报文流
  * 每条报文都分配一个顺序 ID，称为偏置
* 主题是分区（物理文件）的逻辑集合。
   * 有序
   * 仅追加写入
   * 不可变更
* 仅保证主题分区内的排序。
* 生产者发送到特定主题分区的报文保持发送时的顺序。
* 消费者实例查看报文的顺序是报文在日志中存储的顺序。
* 主题分区有备份：容错
* broker 代理包含某个主题的一些分区。
* 一个broker是一个分区的领导leader：所有的写入和读取都必须到leader

##### Kafka Architecture

Coordination

Kafka uses Zookeeper for the following tasks:

Detecting the addition and the removal of brokers and consumers.

Keeping track of the consumed offset of each partition.

##### State in Kafka

**Brokers are sateless**: no metadata for consumers-producers in brokers.

Consumers are responsible for keeping track of **offsets**.

Messages in queues **expire** based on pre-configured **time periods** (e.g., once a day).  

Kafka guarantees that **messages from a single partition are delivered to a consumer in order**. **at-least-once**

### Summary

## 2023-09-22 