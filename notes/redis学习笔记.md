Redis 学习笔记


# NoSQL
> [What is a NoSQL database?](https://redis.com/nosql/what-is-nosql/)

- "no SQL" 或 "not only SQL"，非关系型数据库
- 适合高性能需求环境，读写数据很快
- 通常应用于分布式集群环境中



## RDBMS VS. NoSQL
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


- MoSQL
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



对于 distributed data store，仅能提供下面三种中的两种保障：
- Consistency
强一致性
所有的客户端访问在相同时间访问所有节点得到的数据相同
因此，如果有一个节点的数据发生写操作，必须将其改变同步到其他全部节点才认为写操作成功


- Availability
可用性
如果一个或多个节点出现故障，客户端访问其他节点时不受影响，所有工作的节点必须能得到有效的相应
客户端得到的响应不能有延迟，不能返回错误，但不保证响应的结果是最近写入的数据（如其他节点有写入数据但还未同步到其他节点就出故障）


- Partition tolerance
分区容忍性
分布在不同位置的节点由于网络的原因造成数据丢失，无法通信时仍能继续提供服务



在分布式系统中，P 很容易发生，此时需要在 C 和 A 之间做出选择，根据实际的业务场景，可以分为下面三类：
- CA
单点集群

- CP
如 Zookeeper，PXC 集群

- AP
很多分布式系统的选择
如退款业务，可以延迟一些到账


# Redis
> []()


key-value


较少数据
持久化


## 单线程
处理用户请求是一个线程
一次只运行一条命令，因此需要避免执行较慢的指令


还有其他线程，如做持久化的线程，主从复制等线程


- 纯内存
放少量数据
- 非阻塞



多实例：多个实例


## Redis 常见应用场景
- 缓存
网站

- Session 共享
Web集群中的多服务器间的 session 共享

- 计数器
商品访问排行榜、浏览器、粉丝数、点赞、评论等

- 社交
- 

## 缓存

### 缓存穿透


### 缓存击穿


### 缓存雪崩


taskset


## 慢查询
只计算执行命令的时间，不包括输出指令的时间


## Redis 持久化

### RDB
二进制文件
save 保存的数据有哪些？
添加的 key 等，修改的密码，配置等不会保存

save 执行会阻塞，不保存完，则其他命令无法执行，单线程，save 时用的原来的线程

bgsave fork 一个子进程执行备份，后台保存，备份时生成一个 temp-进程号.rdb 文件，备份完成后覆盖原来的 dump.rdb，防止保存的过程中出故障，因此在保存完后再覆盖

保存当前的快照，当前的结果，二进制文件，不记录过程



### AOF
相当于增量备份

先做一次完全备份，.rdb 文件，后续再做增量备份

优先级高于 RDB 

开启后，

如果当前有数据，先在 redis 中用命令 `configure set appendonly on` 将当前数据保存，然后再改配置文件，这样重启 redis 后能恢复原来的数据

### AOF



### AOF rewrite
