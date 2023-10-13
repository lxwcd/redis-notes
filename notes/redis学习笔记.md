Redis 学习笔记

# 资源
> Redis 设计与实现
> [Redis 官网](https://redis.io/docs/)
> [图解 Redis](https://www.xiaolincoding.com/redis/)


# NoSQL
> [What is a NoSQL database?](https://redis.com/nosql/what-is-nosql/)

- "no SQL" 或 "not only SQL"，非关系型数据库
- 适合高性能需求环境，读写数据很快
- 通常应用于分布式集群环境中


## RDBMS VS NoSQL
> [SQL vs. NoSQL](https://redis.com/nosql/what-is-nosql/#why-use-nosql)

- RDBMS: Relational Database Management System

1. Data Model
- RDBMS 
高度组织化结构化数据
predefined schema 


- NoSQL
more flexible data model
有多种数据模型，如 key-value pairs，documents, columnar or graph structures

2. Scalability
- RDBMS
更适合垂直扩展


- NoSQL
NoSQL 设计就用于分布式存储和处理大规模数据的，更适合水平扩展
They employ distributed architectures that facilitate horizontal scaling by allowing data to be partitioned and distributed across multiple nodes. 
This enables them to handle large amounts of data and accommodate increasing workloads more easily than traditional relational databases like MySQL.


3. Schema
- RDBMS
fixed schema
对 schema 的修改更复杂和耗时


- NoSQL
schema-flexible
allowing for dynamic schema changes without requiring modifications to all existing data


4. Query Language
- RDBMS
使用 SQL 语句

- NoSQL
针对不同的 data model 使用不同的查询语句
less standardized and more specific to the database technology


5. Uses Cases
- RDBMS
RDBMS is widely used in applications that require structured data, transactions, and ACID (Atomicity, Consistency, Isolation, Durability) properties. It is suitable for applications like financial systems, inventory management, and data analysis with complex joins.

适合金融业务等，需要事务，ACID 特性
需要进行复杂 join 查询的数据分析业务
高并发下读写性能较差
不适合非结构化数据存储


- NoSQL
处理大量的结构和非结构化数据
They are commonly used in web applications, real-time analytics, content management systems, and IoT applications.

不适合需要 join 等复杂查询的业务
不适合需要事务支持的业务


## NoSQL 数据库分类
> [NoSQL](https://en.wikipedia.org/wiki/NoSQL)
> [Type of NoSQL databases](https://redis.com/nosql/what-is-nosql/#types-of-nosql-databases)


# CAP 定理
> [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem)
> [What is the CAP theorem?](https://www.ibm.com/topics/cap-theorem)
> [CAP Theorem for System Design - Explained](https://www.turing.com/kb/cap-theorem-for-system-design)


对于 distributed data store，仅能提供下面三种中的两种保障：
- Consistency
强一致性
所有的客户端访问在相同时间访问所有节点得到的数据相同
因此，如果有一个节点的数据发生写操作，必须将其改变同步到其他全部节点才认为写操作成功

- Availability
可用性
如果一个或多个节点出现故障，客户端访问其他节点时不受影响，所有工作的节点必须能得到有效的响应
客户端得到的响应不能有延迟，不能返回错误，**但不保证响应的结果是最近写入的数据**（如其他节点有写入数据但还未同步到其他节点就出故障）

- Partition tolerance
分区容忍性
分布在不同位置的节点由于网络的原因造成数据丢失，无法通信时仍能继续提供服务
Partition tolerance means that the cluster must continue to work despite any number of communication breakdowns between nodes in the system.


***********************

在分布式系统中，P 很容易发生，此时需要在 C 和 A 之间做出选择，根据实际的业务场景，可以分为下面三类：
- CA
单点集群
无分区容忍性

- CP
如 Zookeeper，PXC 集群
When a partition occurs between any two nodes, the system has to shut down the non-consistent node (i.e., make it unavailable) until the partition is resolved.
为了保持一致性，当某个节点出故障时，该节点会被关闭

- AP
很多分布式系统的选择
如退款业务，可以延迟一些到账
When a partition occurs, all nodes remain available but those at the wrong end of a partition might return an older version of data than others.

# Base 属性
> [Base Properties in DBMS](https://www.geeksforgeeks.org/base-properties-in-dbms/)

## Basically Available
基本可用性（Basically Available）是指系统在面对故障或异常情况时仍能够提供基本的服务，即系统不会完全崩溃或不可用。

基本可用性的特点包括：

1. 部分功能限制：系统在故障或异常情况下可能会限制某些功能或服务。这意味着系统可能无法提供完整的功能集，但仍能提供基本的核心功能，以满足用户的基本需求。

2. 性能下降：在故障或异常情况下，系统的性能可能会受到影响。例如，响应时间可能延长、吞吐量可能下降，但系统仍能够在合理的时间范围内提供响应。

3. 部分数据不一致：在故障或异常情况下，系统中的数据可能会出现部分不一致的情况。这是由于异步复制、延迟或冲突解决等原因导致的。系统会尽力处理数据的一致性，但在某个时间点上可能存在一定的不一致性。

为了保证基本可用性，系统可以采取以下策略：

1. 容错和故障处理：系统应具备容错能力，能够检测和处理故障情况。例如，通过使用冗余部件、自动故障切换、节点替换等机制，系统可以保持基本的可用性。

2. 负载均衡：系统可以使用负载均衡机制将请求分散到多个节点上，避免单点故障和过载情况，从而提高系统的可用性和性能。

3. 异步处理和延迟补偿：系统可以采用异步处理的方式，将某些操作延迟执行。这样可以减少对实时性的要求，提高系统的可用性。同时，系统需要提供延迟补偿机制，确保在后续的时间段内将数据副本调整为一致状态。

需要注意的是，基本可用性并不意味着系统可以无限制地牺牲一致性和可用性。它是在特定条件下的权衡选择，适用于一些对一致性要求相对较低、但对高可用性和性能要求较高的应用场景。

总而言之，基本可用性在BASE模型中指的是系统在面对故障或异常情况时仍能够提供基本的服务。为了保证基本可用性，系统可以采取容错和故障处理、负载均衡、异步处理和延迟补偿等策略。这些策略可以帮助系统在故障发生时继续运行，并尽快恢复正常状态。

## Soft State
软状态（Soft state）是指系统中的数据副本可能存在一定的时间窗口内的不一致性。
与ACID模型中的强一致性不同，软状态容许系统中的数据副本在某个时间点上可能处于不一致的状态。

软状态的概念源于分布式系统中的异步复制和最终一致性的设计。
在分布式系统中，数据的写操作可能在不同节点上进行异步复制，导致数据副本之间的延迟和不一致性。这种不一致性在某个时间点上是可以接受的，只要最终系统能够达到一致状态即可。

软状态的特点包括：

1. 延迟：由于数据异步复制的特性，系统中的数据副本之间可能存在一定的延迟。这意味着在某个时间点上，不同节点上的数据副本可能不同步，处于不一致的状态。

2. 容忍不一致性：软状态容许系统在某个时间窗口内存在不一致的数据副本，但要求系统最终能够达到一致状态。通过一致性协议和冲突解决机制，系统会逐渐将数据副本调整为一致。

3. 最终一致性：软状态是最终一致性的一种表现形式。最终一致性指的是系统中的数据副本在一段时间后，最终会达到一致的状态。系统会通过后台的数据同步和冲突解决机制来逐渐调整数据副本，使其达到一致性。

软状态的设计可以带来一些好处，如提高系统的可用性和性能。它允许系统在分布式环境中进行异步操作和复制，减少了同步等待的开销，提高了系统的吞吐量。然而，软状态也带来了一定的挑战，例如处理数据冲突、解决最终一致性等方面的复杂性。

总结而言，软状态在BASE模型中指的是系统中的数据副本可能在某个时间窗口内处于不一致的状态，但要求系统最终能够达到一致性。这种设计可以提高系统的可用性和性能，但也需要考虑数据冲突解决等方面的问题。


## Eventual Consistency
最终一致性（Eventual consistency）是指系统中的数据副本在一段时间后最终会达到一致的状态。与ACID模型中的强一致性不同，最终一致性允许在某个时间段内出现数据副本的不一致，但要求系统最终在合理的时间范围内达到一致性。

最终一致性的特点包括：

1. 异步复制：在分布式系统中，数据的写操作可能在不同节点上进行异步复制。这导致了数据副本之间的延迟和不一致性。最终一致性容许数据副本之间存在一段时间的不一致状态。

2. 延迟和冲突：由于网络延迟、节点故障或并发写操作等原因，系统中的数据副本可能会出现冲突。最终一致性要求系统能够解决这些冲突，并逐渐调整数据副本，使其最终达到一致状态。

3. 逐步调整：最终一致性并不要求系统立即将所有数据副本调整为一致状态。相反，系统通过后台的数据同步和冲突解决机制，逐步调整数据副本，使其最终达到一致性。

最终一致性的实现可以通过多种方式，如版本控制、向量时钟、冲突解决算法等。这些机制可以帮助系统在分布式环境中处理数据冲突和异步复制的问题，逐步达到数据的一致性。

需要注意的是，最终一致性并不意味着系统能够无限期地保持不一致的状态。最终一致性要求系统在合理的时间范围内，通过冲突解决和数据同步等机制，将数据副本调整为一致状态。具体的一致性时间取决于系统设计和业务需求。


# 缓存穿透 缓存击穿 缓存雪崩
> [What is cache penetration, cache breakdown and cache avalanche?](https://www.pixelstech.net/article/1586522853-What-is-cache-penetration-cache-breakdown-and-cache-avalanche)

## 缓存穿透
缓存穿透（Cache Penetration）指的是请求的数据既不在缓存中，也不在数据库中，因此每次请求都要穿透缓存直接查询数据库。
如果频繁发生缓存穿透，会导致数据库负载过高并降低系统性能。

缓存穿透可能是由于恶意攻击或错误的查询导致的。

解决方案：
- 使用布隆过滤器（Bloom Filter）等技术，在缓存层面进行简单的数据过滤，当查询缓存不存在时，通过查询布隆过滤器判断数据是否存在，如果不存在则不会继续查询数据库。
- 对于缓存穿透的查询，可以在缓存中设置一个空值（null）作为标记，避免重复查询数据库。


3. 缓存雪崩（Cache Avalanche）：指的是缓存中的大量数据同时过期或失效，导致大量请求直接访问数据库，造成数据库负载剧增。缓存雪崩通常是由于缓存服务器故障、网络问题或大规模数据失效等情况引起的。

解决方案：
- 设置合适的缓存过期时间，避免大量数据同时过期。
- 使用多级缓存架构，如本地缓存+分布式缓存，减少单点故障的风险。
- 实时监控缓存状态，及时发现并解决缓存服务器故障或网络问题。
- 针对热点数据使用热点数据预热策略，提前加载到缓存中。

综合使用上述解决方案可以有效应对缓存穿透、缓存击穿和缓存雪崩问题，提高系统的稳定性和性能。


## 缓存击穿
缓存击穿（Cache Breakdown）指的是一个热点数据的缓存过期或被删除，此时大量并发请求同时涌入，导致数据库负载激增。
缓存击穿通常发生在缓存中的数据过期时，而该数据又是高频访问的热点数据。

解决方案：
- 在缓存失效前通过后台线程异步更新缓存数据，因此缓存将一直有效
- 使用互斥锁（Mutex）或分布式锁，只允许一个请求去查询数据库并刷新缓存，其他请求等待并使用缓存数据。
- 短暂的缓存屏障（Cache Barrier）：在缓存失效时，可以设置一个短暂的缓存屏障，阻止后续请求直接访问底层存储系统。在屏障期间，只有一个请求能够查询底层存储系统，其他请求等待结果并从缓存中获取数据。
- 异步加载：在缓存失效的情况下，可以使用异步加载机制，让一个请求负责查询底层存储系统并将结果存入缓存，而其他请求等待结果并从缓存中获取数据。


## 缓存雪崩
缓存雪崩（Cache Avalanche）是指在使用缓存系统时，大量缓存数据同时失效或过期，或者缓存服务出故障导致所有相关请求直接访问底层存储系统，造成存储系统负载剧增和性能下降的情况。

解决方案：
- 缓存失效时间随机化：为了避免大量的缓存同时失效，可以将缓存的过期时间进行随机化设置。这样可以避免在某个固定时间点大量缓存同时失效。
- 多级缓存架构：采用多级缓存架构，如本地缓存和分布式缓存的结合，可以减少单点故障和缓存失效的风险。即使某个缓存层发生问题，仍可以从其他缓存层获取数据。
- 异步缓存更新：在缓存失效时，可以使用异步缓存更新机制。当发现缓存失效时，不立即去更新缓存，而是在后台异步地进行缓存的重新填充或更新，以避免大量请求同时访问底层存储系统。
- 限流和熔断机制：对于底层存储系统，可以实施限流和熔断机制，限制并发请求的数量，防止存储系统过载。

# Pipeline
Pipeline 技术是一种用于提高计算机系统性能和效率的方法。它通过将连续的操作划分为多个阶段，并将这些阶段的输出直接传递给下一个阶段，以实现并行处理和流水线化执行。

- 工作原理
Pipeline 技术将一个任务或操作分解为多个独立的阶段，每个阶段执行特定的任务。每个阶段的输出直接传递给下一个阶段，而不需要等待整个任务完成。这样，不同的阶段可以并行执行，从而提高整体的处理速度和效率。

- 并行处理
Pipeline 技术通过并行执行不同阶段的任务，充分利用计算资源，提高系统的并发性和吞吐量。每个阶段在处理当前输入时，前一阶段可以开始处理下一个输入，从而实现并行处理。

- 流水线化执行
Pipeline 技术通过流水线化执行任务，减少了任务执行的等待时间和延迟。当一个阶段完成当前输入的处理后，它可以立即开始处理下一个输入，而不需要等待整个任务的完成。

- 提高系统性能
通过使用 Pipeline 技术，可以将复杂的任务分解为多个简单的阶段，每个阶段专注于特定的任务。这样可以提高系统的吞吐量、响应时间和整体性能。

- 应用领域
Pipeline 技术广泛应用于各种计算密集型和数据密集型任务，例如图像处理、视频编码、编译器优化、数据库查询等。它还在网络通信中用于分组处理和路由选择等方面。

需要注意的是，Pipeline 技术在实践中可能会面临一些挑战，例如处理阶段之间的数据依赖关系、负载均衡、阶段间的同步和通信等。因此，在设计和实现 Pipeline 技术时，需要综合考虑这些因素，以实现高效且可靠的并行处理。

# Redis 简介
> [Introduction to Redis](https://redis.io/docs/about/)

Redis is an open source (BSD licensed), in-memory data structure store used as a database, cache, message broker, and streaming engine. 

Redis provides data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes, and streams. 

Redis has built-in replication, Lua scripting, LRU eviction, transactions, and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.

You can run atomic operations on these types, like appending to a string; incrementing the value in a hash; pushing an element to a list; computing set intersection, union and difference; or getting the member with highest ranking in a sorted set.

To achieve top performance, Redis works with an in-memory dataset. 

Depending on your use case, Redis can persist your data either by periodically dumping the dataset to disk or by appending each command to a disk-based log. 
You can also disable persistence if you just need a feature-rich, networked, in-memory cache.

Redis supports asynchronous replication, with fast non-blocking synchronization and auto-reconnection with partial resynchronization on net split.

Redis is written in ANSI C and works on most POSIX systems like Linux, *BSD, and Mac OS X, without external dependencies. 

Redis 支持多种编程语言

# Redis 特点
> [Introduction to Redis](https://redis.io/docs/about/)

1. 高性能：
Redis主要将数据存储在内存中，因此具有极高的读写性能。它使用了基于内存的数据结构和异步I/O操作，能够实现每秒数十万次的读写操作。

2. 支持多种数据结构
Redis支持多种数据结构，包括字符串、哈希表、列表、集合、有序集合等。这使得Redis不仅仅是一个简单的键值存储系统，还可以应用于更广泛的场景，如缓存、发布订阅、计数器等。

3. 持久化支持
Redis支持数据持久化，可以将内存中的数据保存到磁盘上，以防止数据丢失。它提供了两种持久化方式：RDB快照和AOF日志。RDB快照是将Redis在某个时间点的数据保存到磁盘上，而AOF日志则是将每个写操作追加到日志文件中，以便恢复数据。

4. 分布式支持
Redis提供了集群模式，能够将数据分布在多个节点上，以提高容量和性能。Redis集群使用分片技术将数据分布在多个节点上，并提供了自动数据迁移和故障转移的机制，以保证数据的高可用性。

5. 丰富的功能
除了基本的数据存储和读写操作，Redis还提供了许多丰富的功能，如事务支持、发布订阅机制、Lua脚本执行、过期设置、管道操作等。这些功能使得Redis在各种应用场景下都有很好的灵活性和扩展性。

6. 简单易用
Redis使用简单，具有清晰的命令和API接口，易于学习和使用。它支持多种编程语言的客户端库，开发人员可以方便地与Redis进行交互。

7. 单线程处理用户请求
> [Redis 线程模型](https://xiaolincoding.com/redis/base/redis_interview.html#redis-线程模型)

在大多数情况下，Redis是以单线程方式处理用户请求的。

Redis采用单线程模型是为了避免多线程带来的线程切换开销和锁竞争等问题，从而提高系统的性能和响应速度。
通过单线程模型，Redis可以避免多线程并发访问共享数据的竞争和同步开销。

然而，Redis 程序本身并非单线程，例如，在后台进行持久化操作时，Redis会使用额外的线程进行异步磁盘写入，以避免阻塞主线程的执行。

此外，从Redis 6.0版本开始，引入了多线程模型的实验性支持。这个多线程模型称为RedisGears，用于处理复杂的计算和数据处理任务。RedisGears允许用户编写自定义的Lua脚本，利用多个线程并行执行任务，以提高处理速度。


# Redis 常见应用场景
## 缓存
Redis 性能高，10W QPS (Queries Per Second)

1. 数据库查询缓存：将频繁查询的数据缓存到Redis中，减少对数据库的访问压力。例如，将经常请求的用户配置信息、商品信息或热门文章列表缓存到Redis中，提高读取性能。

2. 页面级缓存：将完整的页面内容或页面片段缓存到Redis中，避免重复渲染和数据库查询。当多个用户请求相同的页面时，可以直接从Redis中获取缓存结果，减少服务器的负载和响应时间。

3. API缓存：将API请求的响应结果缓存到Redis中。例如，将第三方API的响应数据缓存到Redis中，避免频繁请求该API，提高系统的响应速度和可靠性。

4. 用户会话缓存：将用户的会话信息存储在Redis中，在分布式系统中实现会话共享和状态管理。用户登录信息、权限验证令牌等可以存储在Redis中，提供快速的会话访问和验证。

5. 全页缓存：将整个网站或Web应用的静态页面缓存到Redis中，以加速页面加载和提供静态内容。这对于高访问量的网站或内容不经常变化的页面特别有效。

6. 热点数据缓存：将热门或高频访问的数据缓存到Redis中，以提供快速的访问速度。例如，将热门商品的信息、用户关注列表或推荐结果缓存到Redis中，以提高系统的响应性能。

7. 频率限制器：使用Redis的计数器和过期时间等功能实现频率限制。可以限制用户在一定时间内的请求次数，用于防止恶意请求、保护API和资源。

## Session 共享
Web集群中的多服务器间的 session 共享
将用户的会话信息存储在Redis中，在分布式系统和负载均衡环境中很有用。

## 计数器和统计
Redis的原子操作和高性能使其成为计数器和统计数据的理想选择。
它可以用于实时统计在线用户数量、页面点击次数、点赞数等。

## 实时排行榜
Redis的有序集合数据结构和排序功能使其非常适合实时排行榜的构建。
它可以存储和更新用户的得分或指标，并支持快速的排名查询。

## 消息队列
Redis的发布订阅机制和列表数据结构使其成为一个可靠的消息中间件和队列系统。
它可以实现异步任务处理、消息发布和订阅、实时通知等功能。

1. 异步任务处理：将需要异步执行的任务放入Redis的消息队列中，消费者从队列中获取任务并执行。这种方式可以将任务的执行与请求的响应分离，提高系统的并发处理能力和响应速度。

2. 发布/订阅模式：Redis的发布/订阅功能可以用作简单的消息发布和订阅系统。发布者将消息发布到指定的频道，订阅者可以订阅感兴趣的频道并接收相应的消息。

3. 实时通知和推送：将实时通知和推送消息放入Redis的消息队列中，订阅者可以实时接收到相关的通知和推送消息。这在实时聊天、实时更新和实时事件通知等场景中非常有用。

4. 任务调度：使用Redis的消息队列实现任务调度和定时任务。生产者将需要执行的任务放入队列中，消费者按照指定的时间或条件从队列中获取任务并执行。

5. 日志处理：将系统产生的日志消息放入Redis的消息队列中，消费者可以异步地处理和存储日志数据。这可以减轻日志写入对系统性能的影响，并提供灵活的日志处理能力。

6. 数据同步：将需要同步的数据变更操作放入Redis的消息队列中，消费者可以接收到这些变更操作并进行相应的数据同步或更新。

7. 分布式系统协调：Redis的消息队列可以用于分布式系统的协调和通信。不同的服务节点可以通过Redis的消息队列进行消息交换，实现任务分配、状态同步和分布式锁等功能。

需要注意的是，Redis的消息队列是基于发布/订阅模型实现的，而不是像专业的消息队列中间件（如RabbitMQ或Kafka）那样具有高级特性。因此，在选择Redis作为消息队列时，需要根据具体的需求和场景权衡其功能和性能。

## 地理位置
Redis的地理位置数据结构（Geo）可以存储地理坐标和位置信息，并支持空间查询和距离计算，适用于地理位置相关的应用，如附近的人、地理围栏等。

## 分布式锁
Redis提供了分布式锁的实现，可以用于在分布式环境中实现资源的互斥访问和并发控制。

Redis 分布式锁是一种基于 Redis 的机制，用于在分布式系统中实现互斥访问共享资源的功能。它可以确保在多个客户端之间对某个资源进行互斥操作，以防止并发冲突和数据不一致的问题。

以下是 Redis 分布式锁的一些关键特点和使用方法：

1. 互斥性：Redis 分布式锁通过对共享资源的加锁和解锁操作，确保同一时间只有一个客户端能够持有该锁。这样可以避免多个客户端同时对相同资源进行修改或访问，从而保证数据的一致性和正确性。

2. 锁的获取：客户端通过在 Redis 中设置一个特定的键作为锁，可以使用 SETNX（SET if Not eXists）命令或者 RedLock 算法等方式来尝试获取锁。如果锁已经被其他客户端持有，获取锁的操作会失败，需要等待锁的释放。

3. 锁的释放：客户端在对共享资源的操作完成后，通过删除或释放锁的键来释放锁。这样可以让其他客户端继续获取锁并进行操作。

4. 锁的超时处理：为了避免某个客户端在获取锁后发生故障或异常情况，导致锁一直无法释放，可以为锁设置一个超时时间（锁的有效期）。当锁的超时时间到达时，Redis 会自动将该锁释放，以避免死锁情况的发生。

5. 容错性：Redis 分布式锁可以应对网络故障、客户端崩溃等情况，并确保在这些情况下锁的安全性和一致性。一些常见的容错机制如 RedLock 算法使用多个 Redis 实例来增加锁的可靠性和容错性。

需要注意的是，Redis 分布式锁虽然能够提供基本的互斥性，但并不能解决所有的分布式系统并发问题。在使用 Redis 分布式锁时，需要考虑并发操作的正确性、锁的超时设置、锁竞争导致的性能问题等方面的设计和实现细节，以确保分布式锁的有效性和可靠性。

# Redis Pipeline
> [Reids Pipelining](https://redis.io/docs/manual/pipelining/)

Redis pipelining is a technique for improving performance by issuing multiple commands at once without waiting for the response to each individual command. 
Pipelining is supported by most Redis clients. 

Redis is a TCP server using the client-server model and what is called a Request/Response protocol.
客户端通常以阻塞模式发送请求给服务端，即客户端发送请求后一直等待服务端响应，才会继续执行其他任务

RTT (Round Trip Time) 表示在网络通信中发送一个数据包从发送端到接收端，再返回到发送端所经历的时间。


使用 Redis Pipeline 可以将多个命令打包发送到 Redis 服务器，并一次性获取结果，减少了通信次数和解析开销，从而提高了性能。

下面是 Redis Pipeline 的一些关键特点和使用方法：

1. 批量操作：Redis Pipeline 允许一次性发送多个命令到 Redis 服务器，而不需要等待每个命令的响应。这样可以减少往返时间，并提高数据操作的效率。

2. 延迟和吞吐量：通过将多个命令打包在一次请求中，可以减少每个命令的通信延迟。特别是在需要执行大量命令的情况下，Redis Pipeline 可以显著提高吞吐量。

3. 原子性：Redis Pipeline 内部的每个命令仍然是原子执行的，保证了多个命令之间的一致性。这意味着即使在 Pipeline 中的多个命令失败，也不会影响其他命令的执行。

4. 使用方法：Redis Pipeline 的使用方法相对简单。首先，客户端将要执行的多个命令按顺序添加到 Pipeline 中，然后一次性发送给 Redis 服务器。最后，客户端可以一次性获取所有命令的执行结果。

需要注意的是，Redis Pipeline 适用于批量操作或需要批量获取结果的场景。对于单个操作或需要即时获取结果的场景，直接发送单个命令可能更为合适。

在实际应用中，可以根据具体的需求和性能要求，合理选择是否使用 Redis Pipeline 来优化数据操作的效率。

# 安装 Redis
> [Install Redis on Linux](https://redis.io/docs/getting-started/installation/install-redis-on-linux/)


# 客户端程序 redis-cli 连接服务器
> [Redis CLI](https://redis.io/docs/ui/cli/)

Redis 命令行接口 redis-cli

默认客户端连接到服务器的地址为 127.0.0.1，端口为 6379，可以通过 `-h` 和 `-p` 分别指定地址和端口
初始默认无密码，因此直接输入 `redis-cli` 即可登录

如果有密码，可以通过 `-a` 指定密码


# Redis 配置
> [Redis configuration file example](https://redis.io/docs/management/config-file/)

环境：ubuntu 22.04  redis 6

## 配置文件配置
`dpkg -L redis-server` 查看配置文件的位置 `/etc/redis/redis.conf`
### 基本配置
- 默认监听地址为本机
```bash
bind 127.0.0.1 ::1
```
- 默认端口为 6379
- protected_mode 保护模式
默认 yes，如果其他主机的客户端没有认证配置则不能连接，需要密码验证
在 docker 中可以设置为 no
- tcp-backlog tcp 全连接队列长度
默认 511
不能超过内核设置的值 /proc/sys/net/core/somaxconn 和 tcp_max_syn_backlog
- timeout
客户端连接到服务端后，空闲多长时间超时，默认 0 表示永不超时
- tcp-keepalive 
tcp 会话保持时间，默认 300s
- daemonize
默认为 yes，则 redis-server 以守护进程方式后台运行
后天运行时会写一个 pid 文件
- pidfile 
pid 文件路径默认 pidfile /var/run/redis/redis-server.pid
- loglevel
日志级别，默认 notice
- logfile
日志文件位置，默认 /var/log/redis/redis-server.log
- databases
数据库数量，默认 16

### 快照配置
1. 配置服务器自动间隔一段时间执行 bgsave 命令保存 rdb 文件
格式为 `save <seconds> <changes>`

默认设置：
```bash
save 900 1
save 300 10
save 60 10000
```

则只要三个条件之一满足，即会执行 bgsave：
- 900s 内修改至少 1 次
- 300s 内修改至少 10 次
- 60s 内修改至少 10000 次

2. 其他设置
```bash
# 快照保存失败后能否继续写入 
stop-writes-on-bgsave-error yes

# 是否允许压缩 rdb 文件
rdbcompression yes

# rdb 文件的文件名，新创建的文件因为同名会覆盖旧的文件
dbfilename dump.rdb

# rdb 文件和 aof 文件的存放目录
dir /var/lib/redis
```

### AOF 文件管理
AOF 模式默认未开启
```bash
appendonly no
```
默认使用的是 RDB 持久化，如果 AOF 模式开启，则 AOF 优先级会更高，启动时 redis 会加载 AOF 

```bash
appendfilename "appendonly.aof"

# AOF 持久化策略
# appendfsync always
appendfsync everysec
# appendfsync no
```
AOF 文件的保存路径未特意指明，用的是设置的工作目录 `dir /var/lib/redis`


重写配置：
```bash
# 默认 no 表示重写时新的 aof 不同步，相当于在次期间 appendfsync no
no-appendfsync-on-rewrite no

# 当 aof 文件达到多少时自动进行 aof 重写
# 下面参数为触发自动执行 AOF 重写的条件之一，即 AOF 文件的当前大小相对于最后一次执行 AOF 重写时的大小的比例。
# 默认值为 100，表示当 AOF 文件的当前大小达到或超过最后一次执行 AOF 重写时的大小的 100% 时，会触发自动执行 AOF 重写。
# 这个参数可以调整以控制 AOF 重写的频率
# 0 则不自动重写
auto-aof-rewrite-percentage 100

# 该参数表示触发自动执行 AOF 重写的条件之一，即 AOF 文件的最小大小
auto-aof-rewrite-min-size 64mb
```

```bash
# 是否加载由于某些原因导致尾部异常的 AOF 文件
aof-load-truncated yes
```

### 内存管理

### Redis 慢查询配置
slow log
只计算真正执行命令的时间，不包括和客户端的 I/O 操作时间，即客户端发送命令，服务端响应结果，以及在服务器内部排队的时间

```bash
# 定义时间阈值，单位为 us，默认时间为 1s，即命令执行时间超过 1s 即判断为 slow log
slowlog-log-slower-than 10000

# slowlog 日志保存的条数，日志保存在内存中，设置一个上限，否则占用内存
slowlog-max-len 128
```

## config 命令修改配置
config 命令可以查看当前 redis 的配置，并且在不重启的情况下动态修改 redis 配置
但并非所有的配置都可以动态修改

```bash
127.0.0.1:6379> CONFIG HELP
1) CONFIG <subcommand> arg arg ... arg. Subcommands are:
2) GET <pattern> -- Return parameters matching the glob-like <pattern> and their values.
3) SET <parameter> <value> -- Set parameter to value.
4) RESETSTAT -- Reset statistics reported by INFO.
5) REWR
ITE -- Rewrite the configuration file.
```

查看端口：
```bash
127.0.0.1:6379> CONFIG GET PORT
1) "port"
2) "6379"
```
当前版本不能修改端口：
```bash
127.0.0.1:6379> CONFIG SET port 6378
(error) ERR Unsupported CONFIG parameter: port
```

查看并修改连接密码：
```bash
127.0.0.1:6379> CONFIG GET requirepass
1) "requirepass"
2) ""
127.0.0.1:6379> CONFIG SET requirepass "123"
OK
127.0.0.1:6379> CONFIG GET requirepass
1) "requirepass"
2) "123"
```

查看最大内存使用量：
```cpp
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "0"
```
为 0 表示不限制


可以使用通配符
```bash
127.0.0.1:6379> CONFIG GET slowlog*
1) "slowlog-max-len"
2) "128"
3) "slowlog-log-slower-than"
4) "10000"
```


查看全部配置：
```bash
127.0.0.1:6379> CONFIG GET *
```

# Redis 常用命令
> [commands](https://redis.io/commands/)

## info 查看信息
> [info](https://redis.io/commands/info/)


### server 查看当前节点信息
#### run_id
唯一标识该节点
重启后会变化



### replication 查看主从复制信息
主节点查看：
```bash
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=172.27.0.12,port=6379,state=online,offset=1318142,lag=1
slave1:ip=172.27.0.10,port=6379,state=online,offset=1318279,lag=0
master_failover_state:no-failover
master_replid:b5ea703708bf26937e904d33af4c15b964b98109
master_replid2:b7028bd3c8774074343fcb4f48e9c3fc37cd25d6
master_repl_offset:1318416
second_repl_offset:163799
repl_backlog_active:1
repl_backlog_size:536870912
repl_backlog_first_byte_offset:5820
repl_backlog_histlen:1312597
```
- slave0 中的 `lag=1` 表示该 slave 在 data synchronization 方面落后 master 1 秒钟
- slave0 中的 `offset` 表示该 slave 已经同步的数据偏移量


#### master-replid
master 节点的 replication ID
During replication, the master publishes its replication ID, 
which the slaves use to identify and connect to the correct master.


#### master-replid2
> master_replid2: The secondary replication ID, used for PSYNC after a failover.

当主节点故障，更换新的主节点后，新的主节点会 publish a new reolication ID，即为新的 master-replid
同时将旧的 replication ID，即原来的 master-replid 保存为 master-replid2

原来的从节点（replicas）利用旧的 master-replid 和 offset 来与 master 节点同步数据时，
新的 master 节点对比 offset 后，如果 offset 在有效范围内，则只用将相差的部分数据同步给从节点，
该过程即为 `PSYNC`，即 partial synchronization

```bash
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:172.27.0.11
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:2094881
slave_repl_offset:2094881
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:b5ea703708bf26937e904d33af4c15b964b98109
master_replid2:b7028bd3c8774074343fcb4f48e9c3fc37cd25d6
master_repl_offset:2094881
second_repl_offset:163799
repl_backlog_active:1
repl_backlog_size:536870912
repl_backlog_first_byte_offset:5820
repl_backlog_histlen:2089062
```

#### slave_read_repl_offset
> slave_read_repl_offset: The read replication offset of the replica instance.

This refers to the read replication offset of the replica instance. 
It is the offset value representing the last byte position that the replica has successfully read from the master. 
The replica continuously reads data from the master and updates this offset to track its progress in replicating the master's data.

从节点从主节点接受并成功读取的数据的 offset

#### slave_repl_offset
> slave_repl_offset: The replication offset of the replica instance

This represents the replication offset of the replica instance. 
It is the offset value denoting the last byte position that the replica has received and successfully applied. 
The replica receives data from the master and applies it to its own dataset. 
The slave_repl_offset tracks the progress of the applied data, indicating the position up to which the replica has synchronized with the master.


从节点将从主节点读取的数据成功写入到自己的 dataset 中的数据 offset

#### replica_announced
> replica_announced: Flag indicating if the replica is announced by Sentinel.

为 `1` 表示真，该从节点被 sentinel 识别到，以后可以作为 failover 的备选节点使用



#### repl_backlog_size
指 master 节点中 backlog buffer 的最大尺寸，配置文件中有设置该大小：
```bash
# Set the replication backlog size. The backlog is a buffer that accumulates
# replica data when replicas are disconnected for some time, so that when a
# replica wants to reconnect again, often a full resync is not needed, but a
# partial resync is enough, just passing the portion of data the replica
# missed while disconnected.
#
# The bigger the replication backlog, the longer the replica can endure the
# disconnect and later be able to perform a partial resynchronization.
#
# The backlog is only allocated if there is at least one replica connected.
#
repl-backlog-size 512mb
```

backlog buffer 是一个环形的 buffer，用来存放 replication stream sent by master node to 
the replica nodes

该 backlog buffer 位于内存中，如果 buffer 满了会覆盖旧的数据，而如果被覆盖的数据还未
同步到从节点，则不能用 partial synchronization，只能用全量复制

例如，某个从节点出故障，但 master 节点正常，从节点恢复正常后可能数据与 master 节点差了很多，
此时如果从节点未同步的数据仍在 master 节点的 backlog buffer中，则仍可以继续用部分同步（PSYNC），
否则，如果 backlog buffer 的缓冲区已满，部分数据已被覆盖，则只能用 full synchronization


#### repl_backlog_histlen
> repl_backlog_histlen: Size in bytes of the data in the replication backlog buffer

backlog buffer 中实际的数据大小


#### repl_backlog_first_byte_offse
> repl_backlog_first_byte_offset: The master offset of the replication backlog buffer

It represents the starting point of the replication backlog buffer. 


## SLOWLOG 查看慢查询日志
> [SLOWLOG](https://redis.io/commands/?name=slowlog)

SLOWLOG 有三个子命令
```bash
127.0.0.1:6379> SLOWLOG help
1) SLOWLOG <subcommand> arg arg ... arg. Subcommands are:
2) GET [count] -- Return top entries from the slowlog (default: 10).    Entries are made of:
3)     id, timestamp, time in microseconds, arguments array, client IP and port, client name
4) LEN -- Return the length of the slowlog.
5) RESET -- Reset the slowlog.
```

### 查看慢查询记录数目
```bash
127.0.0.1:6379> CONFIG SET slowlog-log-slower-than 1
OK
127.0.0.1:6379> CONFIG GET slowlog*
1) "slowlog-max-len"
2) "128"
3) "slowlog-log-slower-than"
4) "1"
127.0.0.1:6379> SLOWLOG LEN
(integer) 2
```

不指定记录数目，则默认显示最近 10 条

### 查看满查询日志信息
```bash
127.0.0.1:6379> SLOWLOG GET 2
1) 1) (integer) 5
   2) (integer) 1697116648
   3) (integer) 1
   4) 1) "SLOWLOG"
      2) "len"
   5) "127.0.0.1:42914"
   6) ""
2) 1) (integer) 4
   2) (integer) 1697116635
   3) (integer) 62
   4) 1) "SLOWLOG"
      2) "GET"
      3) "1"
   5) "127.0.0.1:42914"
   6) ""
```

上面命令显示最近 2 条慢查询日志，每个日志有 6 个信息
- 慢查询日志编号，递增，总共 6 个日志记录，最后一个编号为 5
- 命令被执行的时间戳
- 日志执行的时长，单位为 us
- 执行的命令，多个则分开多行，如最后一个日志执行的命令为 `SLOWLOG LEN`
- 执行命令的客户端 IP 地址和端口
- 执行命令的客户端名称
可以通过下面命令查看
```bash
127.0.0.1:6379> CLIENT GETNAME
(nil)
```

### 情况慢查询日志
```bash
127.0.0.1:6379> SLOWLOG RESET
OK
127.0.0.1:6379> SLOWLOG LEN
(integer) 0
```


# Redis 持久化
> 官方文档：[Redis persistence](https://redis.io/docs/management/persistence/)
> 官方博客：[Redis persistence demystified](http://oldblog.antirez.com/post/redis-persistence-demystified.html)

持久化指将数据持久化保存，redis 提供下面几种持久化方案：
- RDB
- AOF
- No persistence
You can disable persistence completely. This is sometimes used when caching.
- RDB + AOF

## RDB
- Redis Database
- RDB persistence performs point-in-time snapshots of your dataset at specified intervals.
- 存放保存那一刻的快照，保存快照的时间可以用户自己定义
如果每 2 分钟保存一次快照，则最多丢失 2 分钟的数据
- 快照为 .rdb 文件，默认每次新保存的快照覆盖旧的快照，只有一个快照文件
- RDB 保存的是数据库中的键值对数据

优点：
- 压缩的二进制文件，可以通过该文件恢复保存那一刻数据库的状态
- RBD 文件保存是父进程通过 fork 生产子进程执行，因此不会占用父进程执行磁盘 I/O 操作
- 与 AOF 相比，恢复数据更快
- On replicas, RDB supports partial resynchronizations after restarts and failovers

缺点：
- 当数据库很大时，fork 子进程可能很消耗时间
- 如果设置每间隔 5 分钟备份数据，则可能丢失 5 分钟的数据

### RDB 快照的保存方式
> [RDB 快照是如何实现的呢？](https://xiaolincoding.com/redis/base/redis_interview.html#rdb-快照是如何实现的呢)

rdb 文件都是由 rdbSave 函数完成，只是调用方式不同

#### save 
> [save](https://redis.io/commands/save/)


save 命令是同步保存 rdb 文件，会阻塞主进程直到文件创建完成
redis 6 版本后使用多线程，默认处理客户端命令的还是一个主线程，这里 save 命令阻塞的是主线程还是主进程和多线程的实现方式有关
（多线程是在用户空间管理还是由内核管理）

#### bgsave
> [bgsave](https://redis.io/commands/bgsave/)

bgsave 命令则是异步保存 rdb 文件，主进程 fork 一个子进程后即返回，子进程来保存文件，主进程继续执行其他命令

bgsave 命令执行期间，客户端发送 save 或 bgsave 命令会被服务器拒绝，因为两个命令都是调用 rdbSave 函数，防止产生竞争条件

bgsave 命令执行期间，客户端发送 bgrewriteaof 命令会被拒绝，防止两个子进程同时执行大量的磁盘写入操作


### RDB 文件的载入
rdb 文件是在服务器启动时自动执行的，没有专门的载入 rdb 文件的命令，只要 Redis 服务器启动时检测到 rdb 文件，则自动载入
但如果有 AOF 文件，则载入 AOF 文件而非 RDB 文件，AOF 文件的优先级更高

rdb 文件由 rdbLoad 函数完成

服务器在载入 rdb 文件期间，会一直处于阻塞状态


## AOF
> [persistence](https://redis.io/docs/management/persistence/)

- Append Only File
- 保存的是 Redis 服务器执行的写命令来记录数据库状态
- AOF 功能开启后，第一次会做一个完全备份，备份所有数据，后续再进行增量备份，即备份上次备份后新更新的写指令
- AOF 功能默认未开启，如果开启，则优先级比 RDB 高，服务器启动时会载入 AOF 文件
如果已有数据，保存的是 rdb 文件，然后修改配置文件 `appendonly yes`，重启服务，此时服务器不会加载 rdb 文件，
重启服务 `systemctl restart redis-server.service` 后，看到 `/var/lib/redis` 目录下有一个 `appendonly.aof`的空文件，此时利用 `redis-cli` 连接到服务器，查看之前的数据都没了，因为服务器加载的是空的 AOF 文件而非之前保存的 RDB 文件
因此，这种情况需要先用命令修改配置`configure set appendonly on`，动态开启 AOF 后会自动备份数据，进行一次完全备份，在备份完后再修改配置文件重启，后续就会恢复之前的数据了

AOF 优点：
- 更安全，三种同步策略，默认每秒方式最多只丢失 1s 数据
- 用追加记录命令的方式，即使宕机，也不会破坏已存在的内容
- 当 AOF 文件过大时，可以在后台自动重写
- AOF 采用易理解的日志格式记录操作的命令，方便读取文件内容进行分析修改

AOF 缺点：
- AOF 文件通常比 RDB 文件大
- AOF 采用默认的每秒同步策略时比 RDB 备份的方式慢

### AOF 持久化的实现
- AOF 文件的写入分为命令追加，文件写入和文件同步三个步骤

1. 命令追加 append
当 AOF 功能打开（默认未开启），服务器在每执行一条命令后，以协议格式将被执行的写命令追加到服务器的 aof_buf 缓冲区的末尾

2. 文件写入与同步
Redis 服务器进程是一个事件循环，文件事件负责接收客户端的命令请求以及向客户端发送命令回复
时间事件负责执行像 serverCron 函数之类的需要定时允许的函数

每次结束事件循环前，服务器会调用 flushAppendOnlyFile 函数，根据配置的 appendfsync 选项的值决定其写入和同步行为
该参数值默认为 everysec，即将 aof_buf 缓冲区中的内容调用 write 函数写入到操作系统的缓冲区中保存，然后再由一个单独的线程每间隔一秒将缓冲区的数据 fsync 同步到磁盘，因此最多损失 1s 的数据
如果是 always 则每次都将数据同步到磁盘
如果是 no 则每次只写入到操作系统的缓冲区，由操作系统决定何时将其同步到磁盘


### AOF 重写 rewrite
> [Log rewriting](https://redis.io/docs/management/persistence/)
> [AOF 日志是如何实现的？](https://www.xiaolincoding.com/redis/base/redis_interview.html#aof-日志是如何实现的)

由于 AOF 持久化是通过保存执行的写命令，时间长了则会造成 AOF 文件过大，因此 Redis 提供 AOF 文件重写功能

Redis 服务器创建一个新的 AOF 文件来替代旧的 AOF 文件，新的文件不包含冗余命令，通常体积小得多

新文件的生成不是通过对旧文件的读取、分析等，而是读取服务器当期数据库的状态实现

配置文件中可以设置当 aof 文件大小达到某个值时自动执行重写

重写过程：
- 服务器主进程创建一个子进程 bgrewriteaof 通过f_rewrite 函数进行重写操作
如果用子线程执行，则在修改共享数据时，需要加锁，降低性能
- 为了防止重写时主进程修改数据，导致不一致，Redis 服务器设置了一个 AOF 重写缓冲区
当主进程执行写命令后，将该命令同时发送到 aof_buf 和 aof_rewrite_buf 中
- 当子进程完成重写后，像父进程发送一个信号，父进程接收信号后调用一个信号处理函数
信号处理函数会将 aof_rewrite_buf 中的数据写到新的 AOF 文件中，然后对新的 AOF 文件改名，以原子的操作覆盖旧的 AOF 文件
- 信号处理函数执行完后，父进程继续接收命令请求
信号处理函数执行过程中会阻塞父进程


```bash
Fsync() also blocks the process for all the time needed to complete the write, and if this is not enough, on Linux it will also block all the other threads that are writing against the same file.
```

#### bgrewriteaof 手动执行重写 AOF 命令
> [bgrewriteaof](https://redis.io/commands/bgrewriteaof/) 

The rewrite will be only triggered by Redis if there is not already a background process doing persistence.
- If a Redis child is creating a snapshot on disk, the AOF rewrite is scheduled but not started until the saving child producing the RDB file terminates.
- If an AOF rewrite is already in progress the command returns an error and no AOF rewrite will be scheduled for a later time.
- If the AOF rewrite could start, but the attempt at starting it fails (for instance because of an error in creating the child process), an error is returned to the caller.


# Redis 用户密码管理 ACL
> [acl](https://redis.io/docs/management/security/acl/)

ACL 可以指定特定用户的权限，如对一些设置权限，不让其使用一些危险命令等

- 默认有一个用户 `default`  
```bash
127.0.0.1:6379> ACL USERS
1) "default"
127.0.0.1:6379> ACL list
1) "user default on #8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92 ~* &* +@all"
```

上面的符号标识的权限为：
> to access every possible key (~*) and Pub/Sub channel (&*), 
> and be able to call every possible command (+@all)


- default 用户的密码设置可以在配置文件中的 `requirepass` 中指定
```bash
# IMPORTANT NOTE: starting with Redis 6 "requirepass" is just a compatibility
# layer on top of the new ACL system. The option effect will be just setting
# the password for the default user. Clients will still authenticate using
# AUTH <password> as usually, or more explicitly with AUTH default <password>
# if they follow the new protocol: both will work.
#
# The requirepass is not compatible with aclfile option and the ACL LOAD
# command, these will cause requirepass to be ignored.
#
# requirepass foobared
requirepass 123456
```  
这种方式是为了兼容旧版本，通过 `redis-cli` 连接服务器后，仍要用 `AUTH [username] password` 方式输入密码验证
```bash
127.0.0.1:6379> auth [username] password
```

- 如果使用 ACL 文件，则 `requirepass` 会被忽略
默认未开启该选项
```bash
# Using an external ACL file
#
# Instead of configuring users here in this file, it is possible to use
# a stand-alone file just listing users. The two methods cannot be mixed:
# if you configure users here and at the same time you activate the external
# ACL file, the server will refuse to start.
#
# The format of the external ACL user file is exactly the same as the
# format that is used inside redis.conf to describe users.
#
# aclfile /etc/redis/users.acl
```




# Redis 数据类型
set key value 时怎么设置类型的？







# Redis 主从复制和哨兵
1. 主从结构实现高可用，主节点故障时将从节点提升为主节点
2. 主从结构也可实现负载均衡，从节点可对外提供读操作，但写操作操作只能在主节点
从节点也可设置可写，但只能写临时的数据，如果主节点将数据同步到从节点，则从节点写的数据会丢失
配置文件中默认从节点只读
```bash
# Note: read only replicas are not designed to be exposed to untrusted clients
# on the internet. It's just a protection layer against misuse of the instance.
# Still a read only replica exports by default all the administrative commands
# such as CONFIG, DEBUG, and so forth. To a limited extent you can improve
# security of read only replicas using 'rename-command' to shadow all the
# administrative / dangerous commands.
replica-read-only yes
```
3. 一主多从不是 cluster 模式，默认 redis 运行是 standalone 模式
```bash
# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
# started as cluster nodes can. In order to start a Redis instance as a
# cluster node enable the cluster support uncommenting the following:
#
# cluster-enabled yes
```
4. 默认监听地址为本机，可以修改为监听全部地址
```bash
bind 0.0.0.0 -::1
```
5. 主节点和从节点最好都开启持久化功能
```bash
appendonly yes
```
6. 哨兵是为了自动对 master 和 replica 节点切换，且不影响业务
哨兵可以在主节点出现故障后自动将一个从节点提升为主节点
7. 哨兵机制需要在多个节点上运行同一个 sentinel 进程，是一个分布式系统
哨兵可以在单独的服务器上配置，或者和 redis server 在一起配置
通常需要多个哨兵，且为奇数，即哨兵至少 3 个 
每个 sentinel 进程之间通过流言协议（gossip protocols）来接受 master 进程是否正常
sentinel 进程间通过投票协议（agreement protocols）来决定是执行故障转移，即提升从节点为新 master，
因此 sentinel 节点的数目最好为奇数，配置文件（sentinel 的配置文件）中可以指定当几个哨兵认为 master
节点下线即提升新 master：
```bash
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
#
# Tells Sentinel to monitor this master, and to consider it in O_DOWN
# (Objectively Down) state only if at least <quorum> sentinels agree.
#
# Note that whatever is the ODOWN quorum, a Sentinel will require to
# be elected by the majority of the known Sentinels in order to
# start a failover, so no failover can be performed in minority.
#
# Replicas are auto-discovered, so you don't need to specify replicas in
# any way. Sentinel itself will rewrite this configuration file adding
# the replicas using additional configuration options.
# Also note that the configuration file is rewritten when a
# replica is promoted to master.
#
# Note: master name should not include special characters or spaces.
# The valid charset is A-z 0-9 and the three characters ".-_".
sentinel monitor mymaster 172.27.0.10 6379 2

# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster 123456
```
当一个哨兵判断主节点下线，只是自己的主观判断，即 `SDOWN`，只有达到指定位数的哨兵均判断为 `SDOWN`，
最后才能得出 `ODOWN`，然后更换主节点
哨兵在跟换 master 节点前会进行投票来选出一个 leader，再决定将谁提升为 master 节点
```bash
7:X 01 Jul 2023 15:49:50.236 # +sdown master mymaster 172.27.0.10 6379
7:X 01 Jul 2023 15:49:50.282 * Sentinel new configuration saved on disk
7:X 01 Jul 2023 15:49:50.282 # +new-epoch 1
7:X 01 Jul 2023 15:49:50.286 * Sentinel new configuration saved on disk
7:X 01 Jul 2023 15:49:50.286 # +vote-for-leader 083ae14f7457fc8f64a0fb167017a07db4efd4f7 1
7:X 01 Jul 2023 15:49:50.309 # +odown master mymaster 172.27.0.10 6379 #quorum 3/2
```
8. 哨兵选择新的 master 时，和 redis.conf 中配置的优先级有关，默认的优先级为 100
```bash
# The replica priority is an integer number published by Redis in the INFO
# output. It is used by Redis Sentinel in order to select a replica to promote
# into a master if the master is no longer working correctly.
#
# A replica with a low priority number is considered better for promotion, so
# for instance if there are three replicas with priority 10, 100, 25 Sentinel
# will pick the one with priority 10, that is the lowest.
#
# However a special priority of 0 marks the replica as not able to perform the
# role of master, so a replica with priority of 0 will never be selected by
# Redis Sentinel for promotion.
#
# By default the priority is 100.
replica-priority 100
``` 
9.  sentinel 不是代理而是配置中心





## 主节点配置
主节点也配置 `masterauth`，防止主节点故障后恢复成为从节点

## 从节点配置
从节点的配置文件中需要指明 master 的 ip，端口和密码（如果 master 开启了密码保护）
```bash
# replicaof <masterip> <masterport>

# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the replica to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the replica request.
#
# masterauth <master-password>
```
上面的 `replicaof` 和 `masterauth` 两项 

如果 master 配置了 ACL，还需设置：
```bash
# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the replica to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the replica request.
#
# masterauth <master-password>
```
## 哨兵配置
哨兵有单独的配置文件，编译安装时可以从解压安装包后的目录中找到 `sentinel.conf` 配置文件

必须的配置如下：
```bash
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
#
# Tells Sentinel to monitor this master, and to consider it in O_DOWN
# (Objectively Down) state only if at least <quorum> sentinels agree.
#
# Note that whatever is the ODOWN quorum, a Sentinel will require to
# be elected by the majority of the known Sentinels in order to
# start a failover, so no failover can be performed in minority.
#
# Replicas are auto-discovered, so you don't need to specify replicas in
# any way. Sentinel itself will rewrite this configuration file adding
# the replicas using additional configuration options.
# Also note that the configuration file is rewritten when a
# replica is promoted to master.
#
# Note: master name should not include special characters or spaces.
# The valid charset is A-z 0-9 and the three characters ".-_".
sentinel monitor mymaster 172.27.0.10 6379 2

# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster 123456
#
# Set the password to use to authenticate with the master and replicas.
# Useful if there is a password set in the Redis instances to monitor.
#
# Note that the master password is also used for replicas, so it is not
# possible to set a different password in masters and replicas instances
# if you want to be able to monitor these instances with Sentinel.
#
```
- 指定只从集群的名字，默认为 mymaster，指定初始 master 的 ip 和端口，
最后的 2 为投票时多少哨兵认为 SDOWN 就认为 ODOWN

- 需要指定一个密码来访问 master 和 replicas，最好所有的密码设置一致



## 故障转移过程（failover）

## 主从复制故障原因
1. 硬件和软件配置不一致
如从节点内存比主节点小
主节点用 rename-command,从节点没有该命令





CoW fork: copy-on-write


full synchronization 和 partial synchronization

主节点重启后 master_replid 变化，

master_repl_offset and slave_repl_offset ?

backlog：环形队列，用于复制数据到从节点的缓冲区，

master_replid


多实例用处：分摊流量，防止主节点重启而进行全量备份是一下要复制过多数据

一个机器可以部分主节点，部分从节点，另一个机同样，防止主节点都在一个服务器，而服务器重启


主从复制故障：

repl-diskless-sync-delay and repl-disable-tcp-nodelay difference




# redis cluster

hash slot：怎么知道分配多少 slot ？
一个 slot 并非只能对应一个 key，可能对应多个 key


cluster-require-full-coverage：默认 yes，什么场景需要 yes ？

不支持多 database
从节点不能读和写
不能用 mset 同时设置多个值，不支持 mget

不支持级联

16384 槽位固定？

集群偏斜

集群最少三个节点，最好奇数节点，防止脑裂