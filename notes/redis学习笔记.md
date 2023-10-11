Redis 学习笔记

# 资源


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


高并发：
- 无状态
横向扩展，负载均衡


- 有状态
如 mysql 多主，所有节点数据相同，全局校验
如 redis 多主，每个节点数据不同


# 缓存

## 缓存穿透


## 缓存击穿


## 缓存雪崩


taskset

# Redis 特点
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


# Redis 常见应用场景
- 缓存
网站

- Session 共享
Web集群中的多服务器间的 session 共享

- 计数器
商品访问排行榜、浏览器、粉丝数、点赞、评论等

- 社交
- 



# Redis 慢查询配置
只计算执行命令的时间，不包括输出指令的时间


# Redis 持久化
> [Redis persistence demystified](http://oldblog.antirez.com/post/redis-persistence-demystified.html)



## RDB
- Redis snapshotting
- 存放保存那一刻的快照，保存快照的时间可以用户自己定义
- 如果每 2 分钟保存一次快照，则最多丢失 2 分钟的数据
- 快照为 .rdb 文件，每次新保存的快照覆盖旧的快照，只有一个快照文件


## bgsave
background save
主进程 fork 一个子进程，子进程创建一个 rdb snapshot 作为 temporary file，在临时文件生成完成后，才会通过一个 atomic rename system call 用临时文件替代原来的 rdb 文件



相关配置：`stop-writes-on-bgsave-error no`



二进制文件
save 保存的数据有哪些？
添加的 key 等，修改的密码，配置等不会保存

save 执行会阻塞，不保存完，则其他命令无法执行，单线程，save 时用的原来的线程

bgsave fork 一个子进程执行备份，后台保存，备份时生成一个 temp-进程号.rdb 文件，备份完成后覆盖原来的 dump.rdb，防止保存的过程中出故障，因此在保存完后再覆盖

保存当前的快照，当前的结果，二进制文件，不记录过程



## AOF
相当于增量备份

先做一次完全备份，.rdb 文件，后续再做增量备份

优先级高于 RDB 

开启后，

如果当前有数据，先在 redis 中用命令 `configure set appendonly on` 将当前数据保存，然后再改配置文件，这样重启 redis 后能恢复原来的数据


### AOF rewrite
> [Log rewriting](https://redis.io/docs/management/persistence/)


```bash
Fsync() also blocks the process for all the time needed to complete the write, and if this is not enough, on Linux it will also block all the other threads that are writing against the same file.
```


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



# Redis 常用命令
## info
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
# 安装 Redis

## 包安装




## 编译安装
- 下载安装包并解压
- make 
根据 src/README.md 可以指定安装的路径


- 为二进制文件创建软连接
```bash

```

- 创建配置文件等目录
将源码目录中的配置文件拷贝到安装目录的配置文件路径中


## 优化配置

### overcommit_memory warning 
> 内核支持 overcommit 策略介绍：[overcommit-accounting](https://www.kernel.org/doc/Documentation/vm/overcommit-accounting)

```bash
7305:M 27 Jun 2023 12:22:48.887 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
7305:M 27 Jun 2023 12:22:48.888 * Ready to accept connections
```

根据提示查看内核参数：
```bash
[root@docker docker1]$ whatis sysctl
sysctl (8)           - configure kernel parameters at runtime
sysctl (2)           - read/write system parameters
[root@docker docker1]$
[root@docker docker1]$ sysctl vm.overcommit_memory
vm.overcommit_memory = 0
```

当前内存分配策略为 0，根据提示改为 1


### protected mode
在 docker 中可以设置为 no


## 创建用户和组
```bash
[root@docker etc]$ groupadd -r -g 998 redis
[root@docker etc]$ useradd -g redis -r -s /sbin/nologin -M redis
[root@docker etc]$ id redis
uid=998(redis) gid=998(redis) groups=998(redis)
```






