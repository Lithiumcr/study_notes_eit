---
Author: August
Title: Advanced Internetworking（Amir H. Payberah, et al.）
Classroom: Sal-B & https://kth-se.zoom.us/my/payberah
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

Objective

了解用于存储和处理海量数据的大规模分布式系统。涵盖数据密集型计算平台的各种高级主题，即存储和处理大数据的框架。

预期学习成果 (ILO)
ILO1：了解数据密集型计算平台的主要概念。
ILO2：应用所掌握的知识来存储和处理海量数据。
ILO3：分析数据密集型计算平台的技术优点。

Tasks

review, lab, essay & presentation, project, exam

in group

Grade

Course Material 课程教材

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

* 五个特征 
  * 按需自助服务：消费者可以独立提供计算能力，而无需与服务提供商进行人工交互。
  * 无处不在的网络访问
  * 资源池：提供商的计算资源被集中起来为消费者提供服务
  * 快速弹性：可以快速、弹性地配置功能，在某些情况下可以自动配置。
  * 可衡量的服务：可以监视、控制和报告资源使用情况，为提供商和消费者提供透明度。
* 三种服务模式
  * 基础设施即服务（IaaS）：类似于建造新房子。示例：Amazon Web Services（EC2 实例和 S3 存储）
  * 平台即服务（PaaS）：类似于购买空房子。示例：谷歌应用引擎
  * 软件即服务（SaaS）：类似于住在酒店。示例：Gmail、Github
* 四种部署模型

### Big Data

buzzwords: data characterized by 4 key attributes: **volume, variety, velocity and value.**

Traditional platforms fail to show the expected performance.

### 3-level model

* resource management
* data storage - Distributed File Systems, NoSQL DB, Messaging Systems
* data processing, Batch, Streaming, Graph, Structured Data, ML

Spark Precessing Engine

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

#### GFS Architecture

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

---

* 主机
   * 负责全系统的所有活动
     * 维护所有文件系统元数据：
     * 命名空间、ACL、从文件到块的映射以及块的当前位置
     * 全部保存在内存中，命名空间和文件到块的映射也持久存储在操作日志中
   * 定期与每个 chunkserver 进行通信：
     * 确定块位置
     * 评估整个系统的状态
* 块服务器
   * 管理块
   * 告诉master它有哪些块
   * 将块存储为文件
   * 保持块的数据一致性
* 客户
   * 向主服务器发出控制请求。
   * 直接向块服务器发出数据请求。
   * 缓存元数据。
   * 不缓存数据。

#### Data Flow and Control Flow 

* Data flow is decoupled from control flow 
* Clients interact with the master for metadata operations (control flow) 
* Clients interact directly with chunkservers for all files operations (data flow)

#### Why Large Chunks? (2019)

* 64MB or 128MB (much larger than most file systems)
* Advantages 
  * Reduces the size of the metadata stored in master
  * Reduces clients’ need to interact with master 
* Disadvantages: Wasted space due to internal fragmentation

---

* 64MB 或 128MB（比大多数文件系统大得多）
* 优点
   * 减少master中存储的元数据的大小
   * 减少客主互动的需要
* 缺点：由于内部碎片而浪费空间

### System interactions

System interface

* Not POSIX-compliant, but supports typical file system operations 
  * create, delete, open, close, read, and write
* snapshot: creates a copy of a file or a directory tree at low cost 
* append: allow multiple clients to append data to the same file concurrently

---

* 不符合 POSIX 标准，但支持典型的文件系统操作
   * 创建、删除、打开、关闭、读取和写入
* 快照：以低成本创建文件或目录树的副本
* 追加：允许多个客户端同时追加数据到同一个文件

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
5. 一级节点检查记录是否适合指定的块。
6. 如果记录不适合，则一级节点：
* 填充块，
* 告诉次级也这样做，
* 并通知客户。
* 然后客户端重试附加下一个块。
7. 如果记录符合，则一级节点：
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

主机拥有整个系统的全局知识，简化了设计
（希望）主机永远不会是瓶颈
* 客户端从不通过master读写文件数据
* 客户端仅向 master 请求与哪些 chunkserver 通信
* 对同一块的进一步读取不涉及主机

#### The Master Operations

#### Namespace Management and Locking

### Fault Tolerance

#### For chunks

Chunks replication (re-replication and re-balancing)

Data integrity

* Checksum for each chunk divided into 64KB blocks.
* Checksum is checked every time an application reads the data.  

#### For chunkserver

* All chunks are versioned.

* Version number updated when a new lease is granted.

* Chunks with old versions are not served and are deleted.

---

* 所有块都有版本控制。
* 授予新租约时更新版本号。
* 旧版本的块不会被提供并被删除。

#### For master

* Master state replicated for reliability on multiple machines.
* When master fails:
  * It can restart almost instantly.
  * A new master process is started elsewhere.
* Shadow (not mirror) master provides only read-only access to file system when primary master is down.

---

* 主状态复制以确保多台机器上的可靠性。
* 当master失败时：
   * 它几乎可以立即重新启动。
   * 一个新的主进程在别处启动。
* 当一级主服务器关闭时，影子（非镜像）主服务器仅提供对文件系统的只读访问。

### GFS and HDFS

### Summary

## 2023-09-04 NoSQL Databases
**Database**: an organized collection of data.

**Database Management System (DBMS)**: a software to capture and analyze data.

### Relational SQL DB

SQL is good: 

* Rich language and toolset
* Easy to use and integrate
* Many vendors

#### ACID

1. Atomicity
    All included statements in a transaction are either executed or the whole transaction is aborted without affecting the database.
2. Consistency
    A database is in a consistent state before and after a transaction.
3. Isolation
    Transactions can not see uncommitted changes in the database.
4. Durability
    Changes are written to a disk before a database commits a transaction so that committed data cannot be lost through a power failure.

---

数据库事务的ACID四属性

1. 原子性
    事务中包含的所有语句要么被执行，要么整个事务被中止，而不影响数据库。
2. 一致性
    数据库在事务之前和之后处于一致的状态。
3. 隔离
    事务无法看到数据库中未提交的更改。
4. 耐用性
    在数据库提交事务之前，更改会写入磁盘，以便提交的数据不会因电源故障而丢失。

SQL Databases Challenges: Web-based applications caused spikes.

* Internet-scale data size
* High read-write rates
* Frequent schema changes

**RDBMS were not designed to be distributed.**

### NoSQL

* Avoids:
  * Overhead of **ACID** properties
  * **Complexity** of SQL query
* Provides:
  * **Scalability** 可扩展性
  * Easy and frequent **changes to DB **简单频繁更改
  * **Large data volumes** 大数据量

#### Availability 可用性

Replicating data to improve the availability of data.

Data replication: Storing data in more than one site or node

#### Consistency 一致性

Strong consistency 强一致性

After an update completes, any subsequent access will return the updated value

Eventual consistency 最终一致性
* Does not guarantee that subsequent accesses will return the updated value.
* Inconsistency window.
* If no new updates are made to the object, eventually all accesses will return the last updated value. 如果没有对该对象进行新的更新，最终所有访问都将返回最后更新的值。

The large-scale applications have to be reliable: consistency, availability, partition
tolerance 分区宽容

Achieving ACID properties on large-scale applications is challenging.

#### CAP theorem

**You can choose only two among CAP!**

- C:Consistency，一致性，任何一个读操作总是能够读到之前写操作的结果，也就是所有节点在同一时间有相同的数据
- A:Availablity，可用性，快速获取数据，在确定时间内返回操作结果，不管请求成功或失败都会相应
- P:Tolenrance of Network Partition:分区容忍性，网络出现分区时（系统的节点通信故障），分离的系统也可以正常运行。系统的任意信息丢失不会影响系统的继续运作
- CA：放弃分区容忍性，将所有事务内容放到一台机器上，可扩展性较差。传统关系数据库。
- CP：放弃可用性，出现网络分区影响时，受影响的服务需要等待数据一致，期间无法对外提供服务。BigTable，HBase，Redis，MongoDB等
- AP：放弃一致性，允许系统返回不一致的数据。Dynamo，CouchDB，Riak等

Continue the operation in the presence of network partitions.

#### NoSQL Data Models

##### Key-Value 键值 Data Model (2022)

Collection of KV Pairs

Ordered Key-Value: processing over key ranges.

**Dynamo**, Scalaris, Voldemort, Riak, ...

##### Column-Oriented 列族 Data Model

Similar to a key/value store, but the value can have multiple attributes

Store and process data by column instead of row.

**BigTable, Hbase, Cassandra**, ...

##### Document 文档 Data Model

Similar to a column-oriented store, but values can have complex documents.

Flexible schema (XML, YAML, JSON, and BSON).

CouchDB, MongoDB, ...

##### Graph 图 Data Model

graph structures with nodes, edges, and properties

Neo4J, InfoGrid, ...

---

**1、键值数据库**：Redis,SimpleDB, riak，Chordless

- 优点：扩展性好，灵活性好，写操作性能高
- 缺点：无法存储结构化信息（结构体），条件查询效率低，无法通过值来查询。

**2、列族数据库**：HBase，BigTable

- 优点：查询速度快，可扩展性强，复杂性低
- 缺点：功能较少，不支持事务强一致性，不支持ACID事务

**3、文档数据库**: mongoDB

- 文档：本质上也是键值：HTML，JSON等
- 优点：性能好，高并发，灵活性高，复杂性低，数据结构灵活，既可以通过键来构建索引，也可以根据内容构建索引
- 缺点：缺乏统一的查询语法，不支持文档间的事务

**4、图数据库**: Infinite Graph, Neo4j

- 优点：灵活性高，支持复杂的图算法，可用于构建复杂的关系图谱
- 缺点：复杂性高，只能支持一定的数据规模

### BigTable 大表

* Lots of (semi-)structured data at Google. 
  * URLs, per-user data, geographical locations, ... 
* Distributed multi-level map 分布式多级映射
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

#### Tablet Serving (2018)

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

- **运行速度快**：使用**DAG执行引擎**以支持循环数据流与内存计算

- **容易使用**：支持多种编程语言，也可以使用Spark Shell交互式编程

- **通用性**：Spark提供了完整而强大的**技术栈**

- **运行模式多样**：可运行于独立的集群模式中，可运行于Hadoop中，也可以运行于云环境

- Scala是Spark的主要编程语言，但Spark还支持Java、Python、R作为编程语言

- Scala的优势是提供了**REPL**（Read-Eval-Print Loop，交互式解释 器），提高程序开发效率

- Scala特性：

- - 具备强大的**并发性**（同一时间执行多个任务），支持函数式编程，可以更好地支持分布式系统
  - **语法简洁**，能提供优雅的API
  - 兼容Java，运行速度快，且能融合到Hadoop生态圈中

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

## 2023-09-11 MapReduce 映射规约 (each year)

Scale up or vertically 单元强化

Scale out or horizontally 扩展（多节点）

A shared nothing architecture for processing large data sets with a parallel/distributed algorithm on clusters of commodity hardware. MapReduce takes care of parallelization, fault tolerance, and data distribution; Hide system-level details from programmers. 一种**无共享**架构，用于在**商用硬件集群**上使用**并行/分布式**算法处理大型数据集。MapReduce 负责并行化、容错和数据分布；对程序员隐藏系统级详细信息。

### Programming Model

Data-Parallel Processing

**Map -> Shuffle && Sort -> Reduce** 

example: word count of a file distributed stored

```scala
public static class MyMap extends Mapper<...> {
  private final static IntWritable one = new IntWritable(1);
  private Text word = new Text();
  public void map(LongWritable key, Text value, Context context)
    throws IOException, InterruptedException {
    String line = value.toString();
    StringTokenizer tokenizer = new StringTokenizer(line);
    while (tokenizer.hasMoreTokens()) {
      word.set(tokenizer.nextToken());
      context.write(word, one);
    }
  }
}

public static class MyReduce extends Reducer<...> {
  public void reduce(Text key, Iterator<...> values, Context context)
    throws IOException, InterruptedException {
    int sum = 0;
    while (values.hasNext())
    sum += values.next().get();
    context.write(key, new IntWritable(sum));
  }
}

public static void main(String[] args) throws Exception {
  Configuration conf = new Configuration();
  Job job = new Job(conf, "wordcount");

  job.setOutputKeyClass(Text.class);
  job.setOutputVa
  lueClass(IntWritable.class);

  job.setMapperClass(MyMap.class);
  job.setCombinerClass(MyReduce.class);
  job.setReducerClass(MyReduce.class);

  job.setInputFormatClass(TextInputFormat.class);
  job.setOutputFormatClass(TextOutputFormat.class);

  FileInputFormat.addInputPath(job, new Path(args[0]));
  FileOutputFormat.setOutputPath(job, new Path(args[1]));

  job.waitForCompletion(true);
}
```


Parallelize the data and process;

* Each Map task (typically) operates on a single HDFS block.
* Map tasks (usually) run on the node where the block is stored.
* Each Map task generates a set of intermediate key/value pairs.
* Sorts and consolidates intermediate data from all mappers, after all Map tasks are complete and before Reduce tasks start;
* Each Reduce task operates on all intermediate values associated with the same intermediate key.
* Produces the final output.

---

并行化数据和流程；

* 每个 Map 任务（通常）在单个 HDFS 块上运行。
* Map任务（通常）在存储块的节点上运行。
* 每个Map任务都会生成一组中间键/值对。
* 在所有 Map 任务完成之后、Reduce 任务开始之前，对来自所有 Mappers 的中间数据进行排序和合并；
* 每个Reduce 任务都对与同一中间键关联的所有中间值进行操作。
* 产生最终输出。

Data Flow

#### Mapper

Input: (key, value) pairs 

Output: a list of (key, value) pairs

The Mapper may use or completely ignore the input key.

A standard pattern is to read one line of a file at a time. Key: the byte offset; Value: the content of the line. 

After the Map phase, all intermediate (key, value) pairs are grouped by the intermediate keys, **(key, list of values)** passed to a Reducer.

---

输入：（键，值）对

输出：（键，值）对的列表

映射器可以使用或完全忽略输入键。

标准模式是一次读取文件的一行。键：字节偏移量；值：该行的内容。

在Map阶段之后，所有中间（键，值）对都按中间键分组，**（键，值列表）**传递给Reducer

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

1. 用户程序将输入文件分为M个分片，大小为HDFS区块（64 MB），将它们转换为**键/值对**； 在集群上启动该程序的多个副本。
2. **master**挑选空闲的worker并给每个分配一个map任务或一个reduce任务。
3. 映射工读取相应输入分割的内容。 由**用户定义的映射函数**生成的键/值对缓冲在内存中。
4. 缓冲的对定期写入本地磁盘，划分为 R 个区域（hash(key) mod R）； 本地磁盘上缓冲对的位置被传回主节点，以将这些位置转发给reduce工作节点。
5. **Reduce Worker 从 Map Worker 的本地磁盘远程读取缓冲数据**，然后根据中间键对其进行排序。
6. reduce worker迭代中间数据：对于每个唯一的中间键，它将键和相应的中间值集传递给**用户定义的reduce函数**。 reduce 函数的输出被附加到最终输出文件中。
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

- Spark的设计遵循**“一个软件栈满足不同应用场景”**的理念，逐渐形成了一套完整的生态系统
- 既能够提供内存计算框架，也可以支持SQL即席查询、实时流式计算、机器学习和图计算等
- 可以部署在资源管理器YARN之上，提供一站式的大数据解决方案
- 因此，Spark所提供的生态系统足以应对上述三种场景，即同时支持批处理、交互式查询和流数据处理

**Spark生态系统**

- Spark Core：复杂的批处理
- Spark SQL：基于历史数据的交互式查询
- Spark Streaming：基于实时数据流的数据处理
- MLLib：基于历史数据的数据挖掘（机器学习）
- GraphX：图结构数据的处理

### Spark Applications Architecture 

**1 Driver + some executors** based on the **Cluster manager** 集群管理节点 process

Driver Process 控制进程, SparkSession, main()

* Maintaining information about the Spark application
* Responding to a user’s program or input
* Analyzing, distributing, and scheduling work across the executors

Executor Processes 多个执行进程

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

**directed acyclic graphs (DAG) 有向无环图** data flow, 反映RDD之间的依赖关系

A data flow is composed of any number of **data sources, operators, and data sinks** by connecting their inputs and outputs. 数据流由任意数量的**数据源、运算符和数据接收器**通过连接其输入和输出而组成。

Parallelizable operators

#### Resilient Distributed Datasets (RDD) 弹性分布式数据集

* A distributed memory abstraction
* Immutable collections of objects spread across a cluster, like LinkedList
* An RDD is divided into a number of **partitions**, which are **atomic** pieces of information.
* Partitions of 1 RDD can be stored on different nodes of a cluster
* RDDs were the primary AP* in the Spark 1.x series.
* Virtually all Spark code you run, compiles down to an RDD

---

* 分布式内存抽象模型，分布式存储的抽象概念，提供了一种高受限的共享内存模型
* 分布在集群中的不可变对象集，如同 LinkedList
* RDD可被分为许多**分区**，**原子**的。
* 1个RDD的分区物理上可以存储在集群的不同节点上
* RDD 是 Spark 1.x 系列中的主要 API。
* 实际上，您运行的所有 Spark 代码都会编译为 RDD
* 每个Application都有自己专属的Executor进程，并且该进程在Application运行期间一直驻留。Executor进程以多线程的方式运行Task
* Spark运行过程与资源管理器无关，只要能够获取Executor进程并保持通信即可
* Task采用了数据本地性和推测执行等优化机制

Two types of RDDs: Generic RDD, Key-value RDD

KV RDD have special operations, such as aggregation, and a concept of
custom partitioning by key

- 设计背景

- - 许多迭代式算法（比如机器学习、图算法等）和交互式数据挖掘工具，共同之处是，不同计算阶段之间会重用中间结果
  - MapReduce框架把中间结果写入HDFS，大量数据复制、磁盘IO和序列化开销
  - 为此，RDD提供了一个抽象的数据架构，不必担心底层数据的分布式特性，只需将具体的**应用逻辑表达为一系列转换处理，不同RDD之间的转换操作形成依赖关系**，可以实现**管道化**，避免中间数据存储和磁盘IO消耗

- RDD的概念

- - 分布式对象集合，本质上是一个只读的分区记录集合
  - 每个RDD可分成多个分区，每个分区就是一个数据集片段
  - RDD提供一种高受限的共享内存模型：RDD是只读记录分区的集合，不能直接修改

- Spark使用RDD后为什么能高效计算？

- - 高效的容错机制：不需要冗余复制来容错
  - 中间结果持久化到内存
  - 存放的数据可以是Java对象，避免了不必要的对象序列化与反序列化

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

## 2023-09-22 Lab3 Kafka

## 2023-09-25 Scalable Stream Processing - Spark streaming

Spark streaming, Run a streaming computation as a series of very small, deterministic batch jobs. 

* Chops up the live stream into batches of X seconds.
* Treats each batch as RDDs and processes them using RDD operations.
* Discretized Stream Processing (DStream) 离散流

### DStream

sequence of RDDs representing a stream of data. RDD序列代表数据流

Any operation applied on a DStream translates to operations on the underlying RDDs.

`StreamingContext` is the main entry point of all Spark Streaming functionality.

#### Input Operations

* Socket connection  
* FS
* Connectors with external sources, e.g., Twitter, Kafka, Flume, Kinesis, ...

Transformations on DStreams are still **lazy**! Until computation is kicked off explicitly by a call to the `start()` method

#### Transformations

DStreams support many of the transformations available on normal Spark RDDs

```scala
map
reduce
reduceByKey
```

#### Example - Word Count


```scala
import org.apache.spark._
import org.apache.spark.streaming._
// Create a local StreamingContext with two working threads and batch interval of 1 second.
val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
val ssc = new StreamingContext(conf, Seconds(1))

val lines = ssc.socketTextStream("localhost", 9999)

val words = lines.flatMap(_.split(" "))

val pairs = words.map(word => (word, 1))
val wordCounts = pairs.reduceByKey(_ + _)
wordCounts.print()

// Start the computation
ssc.start()
// Wait for the computation to terminate
ssc.awaitTermination()
```

#### Window Operations

apply to a over a sliding window of data

two parameters: window length and slide interval

tumbling window effect can be achieved by making slide interval = window length

##### Example - Word Count with Window

```scala
val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
val ssc = new StreamingContext(conf, Seconds(1))
val lines = ssc.socketTextStream("localhost", 9999)
val words = lines.flatMap(_.split(" "))
val pairs = words.map(word => (word, 1))
val windowedWordCounts = pairs.reduceByKeyAndWindow(_ + _, Seconds(30), Seconds(10))
windowedWordCounts.print()
ssc.start()
ssc.awaitTermination()
```

#### States

Accumulate and aggregate the results from the start of the streaming job.

Need to check the previous state of the RDD in order to do something with the current RDD  

mandatory: you provide a checkpointing directory for stateful streams.

##### Example - Stateful Word Count

```scala
val ssc = new StreamingContext(conf, Seconds(1))
ssc.checkpoint(".")
//ssc.checkpoint("path/to/persistent/storage")
val lines = ssc.socketTextStream(IP, Port)
val words = lines.flatMap(_.split(" "))
val pairs = words.map(word => (word, 1))
val stateWordCount = pairs.mapWithState(StateSpec.function(updateFunc))

val updateFunc = (key: String, value: Option[Int], state: State[Int]) => {
val newCount = value.getOrElse(0)
val oldCount = state.getOption.getOrElse(0)
val sum = newCount + oldCount
state.update(sum)
(key, sum)
}
```

### Structured Streaming

Treating a live data stream as a table that is being **continuously appended**.

#### Programming Model

Defines a query on the input table, as a static table

Three output modes:

1. **Append**: only the new rows appended to the result table since the last trigger will be written to the external storage.
2. **Complete**: the entire updated result table will be written to external storage.
3. **Update**: only the rows that were updated in the result table since the last trigger will be changed in the external storage  

#### Steps: Define a Streaming Query

#### Streaming Data Sources and Sinks - Files

#### Basic Operations

Most of operations on DataFrame/Dataset are supported for streaming.

#### Window Operations

#### Handling Late Data  

## 2023-09-29 Large scale graph processing

### Challenge

* Difficult to extract parallelism based on partitioning of the data.
* Difficult to express parallelism based on partitioning of computation

### Graph Partitioning

edge-cut, vertex-cut

Natural graphs: skewed Power-Law degree distribution. vertex-cut better

### Example: PageRank Cal. with MapReduce

$$
Rank[i]=\sum_{j\in Nbrs(i)}w_{ji}R[j]
$$

```scala
//map
map(key: [url, pagerank], value: outlink_list)
  for each outlink in outlink_list:
    emit(key: outlink, value: pagerank / size(outlink_list))

  emit(key: url, value: outlink_list)

//reduce
reducer(key: url, value: list_pr_or_urls)
  outlink_list = []
  pagerank = 0

  for each pr_or_urls in list_pr_or_urls:
    if is_list(pr_or_urls):
      outlink_list = pr_or_urls
    else
      pagerank += pr_or_urls

  emit(key: [url, pagerank], value: outlink_list)
```

#### Problems

**MapReduce does not directly support iterative algorithms.** Invariant graph-topology-data **re-loaded and re-processed** at each iteration is wasting I/O, network bandwidth, and CPU  

Materializations of intermediate results at every MapReduce iteration harm performance.

### Pregel: Graph-Parallel Computation

Inspired by **bulk synchronous parallel (BSP)** model

barrier after each iteration

#### Execution Model

Applications run in sequence of iterations, called supersteps.

* A vertex in superstep S can:
  * reads messages sent to it in superstep S-1.
  * sends messages to other vertices: receiving at superstep S+1.
  * modifies its state.
* Vertices communicate directly with one another by sending messages  
* Superstep 0: all vertices are in the active **state**.
* A vertex **deactivates itself by voting to halt**: no further work to do.
* A halted vertex can **be active if it receives a message.**
* The whole algorithm terminates when:
  * All vertices are simultaneously inactive.
  * There are no messages in transit 

| condition          | inactive (State 0) | active   |
| ------------------ | ------------------ | -------- |
| msg rcv & trigger  | active             | active   |
| vote t halt itself | -                  | inactive |

#### Example: PageRank

```scala
Pregel_PageRank(i, messages):
  // receive all the messages
  total = 0
  foreach(msg in messages):
    total = total + msg

// update the rank of this vertex
R[i] = total

// send new messages to neighbors
foreach(j in out_neighbors[i]):
  sendmsg(R[i] * wij) to vertex j
```

### GraphLab / Turi

#### Example: PageRank

```scala
GraphLab_PageRank(i)
// compute sum over neighbors
total = 0
foreach(j in in_neighbors(i)):
total = total + R[j] * wji
// update the PageRank
R[i] = total
// trigger neighbors to run again
foreach(j in out_neighbors(i)):
signal vertex-program on j
```

#### Gather-Apply-Scatter (GAS) 

* Factorizes the local vertices functions into the Gather, Apply and Scatter phases.
* Gather: accumulate information from neighborhood.
* Apply: apply the accumulated value to center vertex.
* Scatter: update adjacent edges and vertices  


```scala
PowerGraph_PageRank(i):
  Gather(j -> i):
    return wji * R[j]

  sum(a, b):
    return a + b
    
// total: Gather and sum
Apply(i, total):
  R[i] = total

Scatter(* -> j):
  if R[i] changed then activate(j)
```

### GraphX: the library to perform graph-parallel processing in Spark

#### The Property Graph Data Model

Unifies data-parallel and graph-parallel systems.

Spark represent graph structured data as a property graph: logically represented as a pair of vertex and edge property collections: VertexRDD and EdgeRDD

Vertex Collection -> Edge Collection -> Triplet Collection -> Building a Property Graph

Graph Operators

### Summary

**Think like a vertex or a table?**

> G. Malewicz et al., “Pregel: a system for large-scale graph processing”, ACM SIGMOD 2010
> Y. Low et al., “Distributed GraphLab: a framework for machine learning and data mining in the cloud”, VLDB 2012
> J. Gonzalez et al., “Powergraph: distributed graph-parallel computation on natural
> graphs”, OSDI 2012
> J. Gonzalez et al., “GraphX: Graph Processing in a Distributed Dataflow Framework”, OSDI 2014

## 2023-10-02 Resource Management - Mesos, YARN, and Borg

schedule resource offering among frameworks:

Running multiple frameworks on a single cluster. 

Maximize utilization and share data between frameworks

Frameworks: Monolithic scheduler vs. two-level scheduler

### Monolithic scheduler

**Global scheduler, use a single, centralized scheduling algorithm for all jobs.**

Advantages

* Can achieve optimal schedule.

Disadvantages

* Complexity: hard to scale and ensure resilience.
* Hard to anticipate future frameworks requirements.
* Need to refactor existing frameworks

### Two-Level Scheduler

**separate concerns of resource allocation and task placement**

Advantages

* Simple: easier to scale and make resilient.
* Easy to port existing frameworks, support new ones.

Disadvantages

* Distributed scheduling decision: not optimal.

### Mesos

a common resource sharing layer, over which diverse frameworks can run.

Computation Model: framework, job, task

Master sends resource offers to frameworks. Frameworks select which offers to accept and which tasks to run. Unit of allocation: resource offer: Vector of available resources on a node

### Fairness: How to allocate resources of different types? (2019, multiple, heterogeneous resources, e.g., CPU, memory, etc)

#### max-min fairness 按需规划

Share guarantee
* Each user can get at least 1/n of the resource.
* But will get less if her demand is less.

Strategy proof
* Users are not better off by asking for more than they need.
* Users have no reason to lie.

#### weighted max-min fairness

#### Asset fairness 资产公平（定比配置）

give weights to resources (e.g., 1 CPU = 1 GB) and equalize total value given to each user. 资源赋权后（将CPU和RAM折算加总）得到总价值，均分给各个用户

**Problem: violates share grantee.**
$$
\max \vec x=(x_1, x_2,...)\\ s.t.
A\vec x\le\vec y,\ A=\matrix{a11, a12\\a21, a22}\\
Value(x_1)=Value(x2)=...=\frac 1n Asset
$$

#### Dominant Resource Fairness, DRF 主控资源公平

*def* Dominant resource of a user: the resource that user has the biggest share of.

Apply max-min fairness to dominant shares: give every user an equal share of her dominant resource. Equalize the dominant share of the users.

用户主控资源：用户占有最大份额的资源。

将最大-最小公平应用于主控份额：**为每个用户提供其（可以是不同类型的）主控资源的平等份额**。即均衡用户的主控份额。

### YARN

Resource Manager (RM) I Application Master (AM) I Node Manager (NM)



### Borg

> Cluster management system at Google



### Docker and Kubernetes



## 2023-10-03 Cloud data lakes

### Evolution of Data Management

#### Data Warehouses 数据仓 (1980s)

**ETL (Extract, Transform, Load)** data directly from operational database systems. 

Purpose-built for **SQL analytics and BI**: schemas, indexes, caching, etc.

Powerful **management features** such as ACID transactions and time travel.

#### Data Lakes 数据湖 (2010s)

Could not support rapidly growing unstructured and semi-structured data: time series, logs, images, documents, etc.

High cost to store large datasets.

No support for data science and ML. 

Low-cost storage to hold all raw data, e.g., Amazon S3, and HDFS.

ETL jobs then load specific data into warehouses, possibly for further ELT.

Directly readable in ML libraries (e.g., TensorFlow and PyTorch) due to open file format.  

---

(2022)

**数据仓**

直接从操作数据库系统中**提取、转换、加载**数据。

专为 SQL 分析和 BI 构建：架构、索引、缓存等。

强大的管理功能，例如 ACID 事务和时间旅行。

**数据湖**

无法支持快速增长的非结构化和半结构化数据：时间序列、日志、图像、文档等。

存储大型数据集的成本很高。

不支持数据科学和机器学习。

低成本存储用于保存所有原始数据，例如 Amazon S3 和 HDFS。

然后，**ETL 作业将特定数据加载到仓库中，可能用于进一步的 ELT。**

由于文件格式开放，可在 ML 库（例如 TensorFlow 和 PyTorch）中直接读取。

#### Problems today

* system architecture
* data reliability
* timelines

Data Lake vs. Data Warehouse

### Lakehouse

### Delta Lake

### Summary