## Redis

### 1. Redis可以用来做什么？

==参考答案==

1. Redis最常用来做缓存，是实现分布式缓存的首先中间件；
2. Redis可以作为数据库，实现诸如点赞、关注、排行等对性能要求极高的互联网需求；
3. Redis可以作为计算工具，能用很小的代价，统计诸如PV/UV、用户在线天数等数据；
4. Redis还有很多其他的使用场景，例如：可以实现分布式锁，可以作为消息队列使用。
5. Redis还可以做注册中心

### 2. Redis和传统的关系型数据库有什么不同？

==参考答案==

​		Redis是一种基于键值对的NoSQL数据库，而键值对的值是由多种数据结构和算法组成的。Redis的数据都存储于内存中，因此它的速度惊人，读写性能可达10万/秒，远超关系型数据库。

​		关系型数据库是基于二维数据表来存储数据的，它的数据格式更为严谨，并支持关系查询。关系型数据库的数据存储于磁盘上，可以存放海量的数据，但性能远不如Redis。

### 3. Redis有哪些数据类型？

==参考答案==

1. Redis支持5种核心的数据类型，分别是字符串（String）、哈希（Hash）、列表（List）、集合（Set）、有序集合（SortedSet）；
   - String 类型的三种格式：字符串、int、float
   - Hash 类型，也叫散列，其value是一个无序字典，类似于 Java 中的 HashMap 结构
   - List 类型与 Java 中的 LinkedList 类似，可以看做是一个双向链表结构
   - Set 类型与 Java 中的 HashSet 类似，可以看做是一个value为null的HashMap
   - SortedSet 类型是一个可排序的 set 集合，与 Java 中的 TreeSet 有些类似，但底层数据结构却差别很大。SortedSet 中的每一个元素都带有一个 score 属性，可以基于 score 属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。
2. Redis还提供了Bitmap、HyperLogLog、Geo类型，但这些类型都是基于上述核心数据类型实现的；
3. Redis在5.0新增加了Streams数据类型，它是一个功能强大的、支持多播的、可持久化的消息队列。

### 4. Redis是单线程的，为什么还能这么快？

==参考答案==

1. 对服务端程序来说，线程切换和锁通常是性能杀手，而单线程避免了线程切换和竞争所产生的消耗；
2. Redis的大部分操作是在内存上完成的，这是它实现高性能的一个重要原因；
3. Redis采用了IO多路复用机制，使其在网络IO操作中能并发处理大量的客户端请求，实现高吞吐率。

关于Redis的单线程架构实现，如下图：

![img](https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/7D358C4626AF51725C251A2611C5DD65.png)



### 5. 如何利用Redis实现一个分布式锁？

==参考答案==

#### 5.1何时需要分布式锁？

​		在分布式的环境下，当多个server并发修改同一个资源时，为了避免竞争就需要使用分布式锁。那为什么不能使用Java自带的锁呢？因为Java中的锁是面向多线程设计的，它只局限于当前的JRE环境。而多个server实际上是多进程，是不同的JRE环境，所以Java自带的锁机制在这个场景下是无效的。

#### 5.2 如何实现分布式锁？

- **采用单点Redis实现分布式锁**。就是在Redis里存一份代表锁的数据，通常用字符串即可。实现分布式锁的思路，以及优化的过程如下：

1. 加锁：

   第一版，这种方式的缺点是容易产生死锁，因为客户端有可能忘记解锁，或者解锁失败。

   ```
   setnx key value
   ```

   第二版，给锁增加了过期时间，避免出现死锁。但这两个命令不是原子的，第二步可能会失败，依然无法避免死锁问题。

   ```redis
   setnx key value 
   expire key seconds
   ```

   第三版，通过“set...nx...”命令，将加锁、过期命令编排到一起，它们是原子操作了，可以避免死锁。

   ```
   set key value nx ex seconds 
   ```

2. 解锁：

   解锁就是删除代表锁的那份数据。

   ```
   del key
   ```

3. 问题：

   看起来已经很完美了，但实际上还有隐患，如下图。线程1在任务没有执行完毕时发生业务阻塞，锁已经超时被释放了，此后不久线程2获取到了锁。等线程1的任务执行结束后，它依然会尝试释放锁，因为它的代码逻辑就是任务结束后释放锁。但是，它的锁早已自动释放过了，它此时释放的就是线程2的锁。

   <img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220601153601412.png" alt="image-20220601153601412"  />

想要解决这个问题，我们需要解决两件事情：

1. 在加锁时就要给锁设置一个标识，线程要记住这个标识。当线程释放锁的时候，要进行判断，是自己持有的锁才能释放，否则不能释放。可以为key赋一个随机值（例如UUID + 线程唯一标识threadId），来充当线程的标识。
2. 释放锁时要先判断、再释放，这两步需要保证原子性，否则第二步失败的话，就会出现死锁。而获取和删除命令不是原子的，这就需要采用Lua脚本，通过Lua脚本将两个命令编排在一起，而整个Lua脚本的执行是原子的。

按照以上思路，优化后的命令如下：

```lua
-- 加锁 set key random-value nx ex seconds   
-- 解锁 
if (redis.call('get',KEYS[1]) == ARGV[1]) then    
    return redis.call('del',KEYS[1]) 
else     
  	return 0 
end
```

- **集群模式**

  > 经过刚刚的讨论，我们已经有较好的方法获取锁和释放锁。基于Redis单实例，假设这个单实例总是可用，这种方法已经足够安全。如果这个Redis节点挂掉了呢？

  到这个问题其实可以直接聊到 Redlock 了。但是你别慌啊，为了展示你丰富的知识储备（疯狂的刷题准备），你得先自己聊一聊 Redis 的集群，你可以这样去说：

  为了避免节点挂掉导致的问题，我们可以采用Redis集群的方法来实现Redis的高可用。

  **Redis集群方式共有三种：主从模式，哨兵模式，cluster(集群)模式**

  - **主从模式**会保证数据在从节点还有一份，但是主节点挂了之后，需要手动把从节点切换为主节点。它非常简单，但是在实际的生产环境中是很少使用的。

  - **哨兵模式**就是主从模式的升级版，该模式下会对响应异常的主节点进行主观下线或者客观下线的操作，并进行主从切换。它可以保证高可用。

  - **cluster (集群)模式**保证的是高并发，整个集群分担所有数据，不同的 key 会放到不同的 Redis 中。每个 Redis 对应一部分的槽。

  （上面三种模式也是面试重点，可以说很多道道出来，由于不是本文重点就不详细描述了。主要表达的意思是你得在面试的时候遇到相关问题，需要展示自己是知道这些东西的，都是面试的套路。）

  > 在上面描述的集群模式下还是会出现一个问题，**由于节点之间是采用异步通信的方式，如果刚刚在 Master 节点上加了锁，但是数据还没被同步到 Salve，这时 Master 节点挂了，它上面的锁就没了，等新的 Master 出来后（主从模式的手动切换或者哨兵模式的一次 failover 的过程），就可以再次获取同样的锁，出现一把锁被拿到了两次的场景。**

  锁都被拿了两次了，也就不满足安全性了。**一个安全的锁，不管是不是分布式的，在任意一个时刻，都只有一个客户端持有。**

- **基于RedLock算法的分布式锁：**

  

  <img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220601155009558.png" alt="image-20220601155009558" style="zoom: 67%;" />

​	

为了解决上面的问题，Redis 的作者提出了名为 Redlock 的算法。该算法基于多个Redis节点，它的基本逻辑如下：

- 这些节点相互独立，不存在主从复制或者集群协调机制；
- 加锁：以相同的KEY向N个实例加锁，只要超过一半节点成功，则认定加锁成功；
- 解锁：向所有的实例发送DEL命令，进行解锁；

RedLock算法的示意图如下，我们可以自己实现该算法，也可以直接使用Redisson框架。

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/5B9C19EC2AFBB10977147566DF33BE03.png" alt="img" style="zoom:50%;" />

参考：

1. *Redis锁从面试连环炮聊到神仙打架* *https://mp.weixin.qq.com/s?__biz=Mzg3NjU3NTkwMQ==&mid=2247505097&idx=1&sn=5c03cb769c4458350f4d4a321ad51f5a&source=41#wechat_redirect*

2. *03-使用Redisson实现RedLock原理* https://cloud.tencent.com/developer/article/1602970

---

### 6.Redis的持久化策略

==参考答案==

Redis支持**RDB持久化**、**AOF持久化**和**RDB-AOF混合持久化**这三种持久化方式。

#### 6.1RDB持久化

**RDB(Redis Database)**是Redis默认采用的持久化方式，它以快照（Snapshot）的形式将进程数据持久化到硬盘中。RDB会创建一个经过压缩的二进制文件，文件以`.rdb`结尾，内部存储了各个数据库的键值对数据等信息。RDB持久化的触发方式有两种：

- **手动触发**：通过 SAVE 或 BGSAVE 命令触发 RDB 持久化操作，创建 “.rdb” 文件；
- **自动触发**：通过配置选项，让服务器在满足指定条件时自动执行 BGSAVE 命令。

其中，`SAVE`命令执行期间，Redis服务器将阻塞，直到`.rdb`文件创建完毕为止。而`BGSAVE`命令是异步版本的`SAVE`命令，它会使用Redis服务器进程的子进程，创建`.rdb`文件。`BGSAVE`命令在创建子进程时会存在短暂的阻塞，之后服务器便可以继续处理其他客户端的请求。总之，`BGSAVE`命令是针对`SAVE`阻塞问题做的优化，Redis内部所有涉及RDB的操作都采用`BGSAVE`的方式，而 SAVE 命令已经废弃！



快照持久化是 Redis 默认采用的持久化方式，在 `redis.conf` 配置文件中默认有此下配置：

```properties
# 在900秒(15分钟)之内，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
save 900 1           
# 在300秒(5分钟)之内，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
save 300 10          
# 在60秒(1分钟)之内，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
save 60 10000        
```

BGSAVE命令的执行流程，如下图：

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/939F2BF55AFF0E48184628AE3B19BB67" alt="img" style="zoom: 50%;" />

BGSAVE 命令的原理，如下图：

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/7195949B05E80D8EEBFE084774FF8753" alt="img" style="zoom: 50%;" />



RDB持久化的优缺点如下：

- 优点：RDB生成紧凑压缩的二进制文件，体积小，使用该文件恢复数据的速度非常快；
- 缺点：BGSAVE每次运行都要执行 fork 操作创建子进程，属于重量级操作，不宜频繁执行，所以 RDB 持久化没办法做到实时的持久化。

#### 6.2 AOF持久化

**`AOF（Append Only File）`**，解决了数据持久化的实时性的问题，是目前Redis持久化的主流方式。AOF的工作流程包括：命令写入（append）、文件同步（sync）、文件重写（rewrite）、重启加载（load）,如下图：

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/72D05C716050D5C022C41B7911FAED6D" alt="img" style="zoom:50%;" />

开启 AOF 持久化后每执⾏⼀条 **写命令**， Redis 就会将该命令写⼊硬盘中
的 AOF ⽂件。 AOF ⽂件的保存位置和 RDB ⽂件的位置相同，

都是通过 dir 参数设置的，默认的
⽂件名是 appendonly.aof 。



AOF默认不开启，需要修改配置项来启用它：

```properties
# 是否开启AOF功能，默认是no
appendonly yes         
# AOF文件的名称
appendfilename "appendonly.aof"
```

在 Redis 的配置⽂件中存在三种不同的 AOF 持久化⽅式，它们分别是：  

```properties
# 每次有数据修改发⽣时都会写⼊AOF⽂件,这样会严重降低Redis的速度
appendfsync always 
# 写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件，是默认方案
appendfsync everysec
# 写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘
appendfsync no
```

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220608191431839.png" alt="image-20220608191431839" style="zoom:67%;" />

AOF持久化的优缺点如下：

- 优点：与RDB持久化可能丢失大量的数据相比，AOF持久化的安全性要高很多。通过使用`appendfsync everysec`选项，用户可以将数据丢失的时间窗口限制在1秒之内。
- 缺点：AOF文件存储的是协议文本，它的体积要比二进制格式的`.rdb`文件大很多。AOF需要通过执行AOF文件中的命令来恢复数据库，其恢复速度比RDB慢很多。AOF在进行重写时也需要创建子进程，在数据库体积较大时将占用大量资源，会导致服务器的短暂阻塞。



##### 补充内容： AOF 重写（rewrite）

​	AOF 重写可以产⽣⼀个新的 AOF ⽂件，这个新的 AOF ⽂件和原有的 AOF ⽂件所保存的数据库
状态⼀样，但体积更⼩。

![image-20210725151729118](https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20210725151729118.png)

​	AOF 重写是⼀个有歧义的名字，该功能是通过读取数据库中的键值对来实现的，程序⽆须对现有
AOF ⽂件进⾏任何读⼊、分析或者写⼊操作。

​	在执⾏ BGREWRITEAOF 命令时， Redis 服务器会维护⼀个 AOF 重写缓冲区，该缓冲区会在⼦进程创建新 AOF ⽂件期间，记录服务器执⾏的所有写命令。当⼦进程完成创建新 AOF ⽂件的⼯作之后，服务器会将重写缓冲区中的所有内容追加到新 AOF ⽂件的末尾，使得新旧两个 AOF ⽂件所保存的数据库状态⼀致。最后，服务器⽤新的 AOF ⽂件替换旧的 AOF ⽂件，以此来完成
AOF ⽂件重写操作  



#### 6.3 RDB-AOF混合持久化：

Redis从4.0开始引入RDB-AOF混合持久化模式，这种模式是基于AOF持久化构建而来的。用户可以通过配置文件中的

`aof-use-rdb-preamble yes`配置项开启AOF混合持久化。Redis服务器在执行AOF重写操作时，会按照如下原则处理数据：

- 像执行BGSAVE命令一样，根据数据库当前的状态生成相应的RDB数据，并将其写入AOF文件中；
- 对于重写之后执行的Redis命令，则以协议文本的方式追加到AOF文件的末尾，即RDB数据之后。通过使用RDB-AOF混合持久化，用户可以同时获得RDB持久化和AOF持久化的优点，服务器既可以通过AOF文件包含的RDB数据来实现快速的数据恢复操作，又可以通过AOF文件包含的AOF数据来将丢失数据的时间窗口限制在1s之内。

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220608191640609.png" alt="image-20220608191640609" style="zoom:67%;" />



### 7.什么是主从模式？（非重点，理解记忆）

> 其实 Redis 的主从模式很简单，在实际的生产环境中很少使用，不建议在实际的生产环境中使用主从模式来提供系统的高可用性，之所以不建议使用都是由它的缺点造成的，在数据量非常大的情况，或者对系统的高可用性要求很高的情况下，主从模式也是不稳定的。虽然这个模式很简单，但是这个模式是其他模式的基础，所以理解了这个模式，对其他模式的学习会很有帮助。

​	单节点Redis的并发能力是有上限的，要进一步提高Redis的并发能力，就需要搭建主从集群，实现<font color="red">**读写分离**</font>。

​	Redis 多机器部署时，这些机器节点会被分成两类，一类是**主节点（Master 节点）**，一类是**从节点（Slave 节点）**。一般主节点进行写操作，而从节点只能进行读操作。当主节点的数据发生变化时，会将变化的数据同步给从节点，这样从节点的数据就可以和主节点的数据保持一致了。一个主节点可以有多个从节点，但是一个从节点会只会有一个主节点，也就是所谓的**一主多从**结构。

![img](https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/fe86a1d3d76c8cccaff47dcd75b04a28.png)

（1）文件配置方式如下：

```properties
# 配置主节点的ip和端口
slaveof 192.168.1.10 6379
# 从redis2.6开始，从节点默认是只读的
Slave -read-only yes
# 假设主节点有登录密码，是123456
masterauth 123456
```

（2）或者通过redis-cli配置

```shell
127.0.0.1:6379> redis-server --slaveof 127.0.0.1 7001 (redis:6379设置为redis:7001的从节点)
```

#### 7.1  Redis的主从同步是如何实现的？

==参考答案==

Redis使用`psync`命令完成主从同步，同步过程分为**全量同步**和**增量同步**。**全量同步**一般用于初次同步的场景，**增量同步**则用于处理因网络中断等原因造成数据丢失的场景。`psync`命令需要以下参数的支持：

1. **`[replicationid]`**：简称 replid，数据集的标记，id 一致则说明是同一数据集。每一个 master 都有唯一的replid，slave 则会继承 master 节点的 replid 。
2. **`[offset]`**：同步偏移量，随着记录在 repl_baklog 中的数据增多而逐渐增大。slave 完成同步时也会记录当前同步的 offset。如果 slave 的 offset 小于 master 的 offset，说明 slave 数据落后于 master，需要更新。

#### 7.2 Redis如何进行全量同步？

流程：

> replicaof 命令等同于 slaveof 命令

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220608194346413.png" alt="image-20220608194346413" style="zoom: 67%;" />

这里有一个问题，Master 如何得知salve是第一次来连接呢？？

有几个概念，可以作为判断依据：

- **`replication id`**：简称replid，是数据集的标记，id一致则说明是同一数据集。每一个Master 都有唯一的replid，Slave 则会继承Master 节点的replid
- **`offset`**：偏移量，随着记录在repl_baklog中的数据增多而逐渐增大。Slave 完成同步时也会记录当前同步的offset。如果Slave 的offset小于Master 的offset，说明Slave 数据落后于Master ，需要更新。

因此Slave 做数据同步，必须向Master 声明自己的replication id 和offset，Master 才可以判断到底需要同步哪些数据。

因为Slave 原本也是一个Master ，有自己的replid和offset，当第一次变成Slave ，与Master 建立连接时，发送的replid和offset是自己的replid和offset。Master 判断发现Slave 发送来的replid与自己的不一致，说明这是一个全新的Slave ，就知道要做全量同步了。

Master 会将自己的replid和offset都发送给这个Slave ，Slave 保存这些信息。以后Slave 的replid就与Master 一致了。

因此，**Master 判断一个节点是否是第一次同步的依据，就是看replid是否一致**。

完整流程描述：

- Slave 节点请求增量同步
- Master 节点判断replid，发现不一致，拒绝增量同步
- Master 将完整内存数据生成RDB，发送RDB到Slave 
- Slave 清空本地数据，加载Master 的RDB
- Master 将RDB期间的命令记录在repl_baklog，并持续将log中的命令发送给Slave 
- Slave 执行接收到的命令，保持与Master 之间的同步

#### 7.3 Redis如何进行增量同步？

全量同步需要先做RDB，然后将RDB文件通过网络传输给Slave ，成本太高了。因此除了第一次做全量同步，其它大多数时候Slave 与Master 都是做**增量同步**。

什么是增量同步？就是只更新Slave 与Master 存在差异的部分数据。如图：

![image-20210725153201086](https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20210725153201086.png)



![image-20220608220422392](https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220608220422392.png)



#### 7.4 什么时候执行全量同步？

- Slave 节点第一次连接Master 节点时
- Slave 节点断开时间太久，`repl_baklog`中的 offset 已经被覆盖时

#### 7.5 什么时候执行增量同步？

- Slave 节点断开又恢复，并且在`repl_baklog`中能找到 offset 时

#### 7.6 如何优化主从同步？

主从同步可以保证主从数据的一致性，非常重要。

可以从以下几个方面来优化Redis主从集群：

- 在Master 中配置`repl-diskless-sync yes`启用无磁盘复制（即直接写入网络流中），避免全量同步时的磁盘IO。
- Redis单节点上的内存占用不要太大，减少RDB导致的过多磁盘IO
- 适当提高`repl_baklog`的大小，发现 Slave 宕机时尽快实现故障恢复，尽可能避免全量同步
- 限制一个 Master 上的 Slave 节点数量，如果实在是太多 Slave ，则可以采用**主-从-从链式结构**，减少 Master 压力

主从从架构图：

![image-20210725154405899](https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20210725154405899.png)



#### 7.7 主从模式的优缺点

##### 优点

- 支持主从复制，Master 能自动将数据同步到 Slave；
- 为了分载 Master 的读操作压力，Slave 服务器可以为客户端提供只读操作的服务，写服务由 Master 来完成，可以进行**读写分离**；
- Slave 同样可以接受其他 Slaves 的连接和同步请求，即可以采用**主-从-从链式结构**，这样可以有效地分载 Master 的同步压力;
- Master 和 Slave 都是以非阻塞的方式完成数据同步。所以在 Master-Slave 同步期间，客户端仍然可以提交查询或修改请求；

##### 缺点

- 不具备自动容错和恢复功能，Master 或 Slave 的宕机都可能导致客户端读写请求失败，需要等待机器重启或者手动切换客户端的 IP 才能恢复；
- Master 宕机，如果宕机前数据没有同步完，则切换IP后会存在数据不一致的问题，降低了系统的可用性；
- 如果多个 Slave 断线了，需要重启的时候，尽量不要在同一时间段进行重启。因为只要 Slave 启动，就会发送 psync 请求和主机全量同步，当多个 Slave 重启的时候，可能会导致 Master IO 剧增从而宕机；
- 难以支持在线扩容，Redis的容量受限于单机配置。



### 8.说⼀下你对哨兵模式的理解？  

​	在主从模式下，Redis 同时提供了哨兵命令`redis-Sentinel `。

哨兵模式是在主从模式的基础上增加了哨兵机制，哨兵作为一个独立的进程运行，其原理是哨兵进程向所有的 Redis 机器发送命令，等待 Redis 服务器响应，从而监控运行的多个 Redis 实例。

​	Sentinel 可以有多个，一般为了便于决策选举，使用奇数个 Sentinel 。Sentinel 可以和 Redis 机器部署在一起，也可以部署在其他的机器上。多个 Sentinel 构成一个 Sentinel 集群， Sentinel 也会相互通信，检查 Sentinel 是否正常运行。如果发现 Master 宕机，Sentinel 之间会进行决策选举新的 Master

哨兵的结构如图：

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220608224829810.png" alt="image-20220608224829810" style="zoom: 67%;" />



#### 9.1 哨兵是如何监控集群的？

Sentinel基于**心跳机制**监测服务状态，每隔1秒向集群的每个 Redis 实例发送`ping`命令：

- **主观下线**：如果某 Sentinel 节点发现某实例未在规定时间响应，则认为该实例**主观下线**。

- **客观下线**：若超过指定数量（quorum）的 Sentinel 都认为该实例主观下线，则该实例**客观下线**。quorum 值最好超过Sentinel 实例数量的一半。

#### 9.2 哨兵的作用如下：

- **监控**：Sentinel 会基于心跳机制监控 Redis 集群中的每一台 Redis 实例的运行状态

- **自动故障恢复**：如果 Master 故障，Sentinel 会选举一个 Slave 提升为 Master。当故障实例恢复后也以新的 Master 为主

- **通知**：Sentinel 充当 Redis 客户端的服务发现来源，当集群发生故障转移时，会将最新信息推送给Redis的客户端

#### 9.3 哨兵是如何完成集群故障恢复的？

一旦发现 Master 故障，Sentinel 需要在 Slave 中选择一个作为新的 Master ，选择依据是这样的：

- 首先会判断Slave 节点与Master 节点断开时间长短，如果超过指定值（down-after-milliseconds * 10）则会排除该Slave 节点
- 然后判断 Slave 节点的`slave-priority`值，越小优先级越高，如果是0则永不参与选举
- 如果`slave-prority`一样，则判断 Slave 节点的 offset 值，越大说明数据越新，优先级越高
- 最后是判断 Slave 节点的运行id大小，越小优先级越高。

#### 9.4 当选出一个新的Master 后，该如何实现切换呢？

流程如下：

- Sentinel 给备选的 Slave 1节点发送`slaveof no one`命令，让该节点成为 Master 
- Sentinel 给所有其它 Slave 发送`slaveof [ip] [port]` 命令，让这些 Slave 成为新 Master 的从节点，开始从新的Master 上同步数据。
- 最后，Sentinel 将故障节点标记为 Slave ，当故障节点恢复后会自动成为新的 Master 的 Slave 节点

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220608230327405.png" alt="image-20220608230327405" style="zoom:67%;" />





#### 9.5 哨兵模式的优缺点

##### 优点

- 基于**主从模式**，拥有主从模式的所有优点，如支持主从复制，能自动进行主从同步；可以进行读写分离；可以采用**主-从-从链式结构**，从而有效地分载 Master 的同步压力；Master 和 Slave 都是以非阻塞的方式完成数据同步，所以在 Master-Slave 同步期间，客户端仍然可以提交查询或修改请求。
- **哨兵模式**下，支持自动故障恢复，系统可用性更高

##### 缺点

- 如果多个 Slave 断线了，需要重启的时候，尽量不要在同一时间段进行重启。因为只要 Slave 启动，就会发送 sync 请求和主机全量同步，当多个 Slave 重启的时候，可能会导致 Master IO 剧增从而宕机；
- <font color="red">难以支持在线扩容，Redis的容量受限于单机配置</font>
- <font color="red">每台机器上的数据是一样的，内存的复用性较低，同时需要额外的资源来启动 Sentinel 进程，实现相对复杂一点</font>



### 10. 说一下你对集群模式（Redis Cluster）的理解？

​	Redis 的哨兵模式基本已经可以实现高可用，读写分离 ，但是在这种模式下每台 Redis 服务器都存储相同的数据，很浪费内存，而且不支持在线扩容，难以解决<u>海量数据存储问题</u>和<u>高并发写的问题</u>。所以在 redis3.0 上加入了 Cluster 集群模式，实现了 Redis 的分布式存储，对数据进行分片，也就是说每台 Redis 节点上存储不同的内容。

#### 10.1 分片集群特征是什么？

> 可以理解为将哨兵与master合为一体了

- 集群中有多个 Master，每个 Master 保存不同数据；每个 Master 又可以有多个 Slave 节点。每个节点都会通过集群总线(cluster bus)，与其他的节点进行通信（完全图）
- Master 之间通过心跳机制`ping`监测彼此健康状态
- Redis-Cluster模式 采用去中心化的思想，没有中心节点的说法，客户端与 Redis 节点直连，不需要连接集群所有节点，连接集群中任何一个可用节点最终都会被转发到正确节点（路由转发）
- 为了保证高可用，Cluster 模式也引入主从复制模式，一个主节点对应一个或者多个从节点。如果半数以上的 Master 与 Master 1 通信超时，那么认为 Master 1 宕机了，就会启用 Master 1 的从节点 Slave 1，将 Slave 1 变成主节点继续提供服务。
- 如果 至少一个Master  和它的所有 Slave 都宕机了，整个集群就会进入 `fail` 状态。因为集群的 slot 映射不完整。如果集群超过半数以上的 Master 挂掉，无论是否有 Slave，集群都会进入 `fail` 状态。

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/64bc0e3e8784bd3830a84be4b699df48.png" alt="img" style="zoom:50%;" />

#### 10.2 说说 Redis 哈希槽的概念？
Redis 集群模式没有使用一致性 hash,而是引入了哈希槽的概念， Redis 集群有 16384 (0~16383) 个哈希槽，每个 key
通过 CRC16 校验后对 16384 取模来决定放置哪个槽，Redis会把每一个master节点映射到16384个插槽（hash slot）上，集群的每个节点负责一部分 hash 槽。  这就意味着Redis 集群最大节点个数是
**16384** 个  

> 一致性hash https://developer.huawei.com/consumer/cn/forum/topic/0203810951415790238?fid=0101592429757310384

#### 10.3 Redis 集群会有写操作丢失吗？为什么？
Redis 并不能保证数据的强一致性，这意味这在实际中集群在特定的条件下可能会丢失写操作。  

#### 10.4 集群扩缩容（集群伸缩）

对 redis 集群的扩容就是向集群中添加机器，缩容就是从集群中删除机器；并重新将 16384 个 slots 分配到集群中的节点上（即数据迁移）。

- **扩容（add-nodes）**时，先使用`redis-cli --cluster add-node`将新的机器加到集群中，这时新机器虽然已经在集群中了，但是没有分配 slots，依然是不起做用的。在使用 `redis-cli --cluster  reshard`进行分片重哈希（插槽转移），将旧节点上的 部分slots 转移到新节点上后，新节点才能起作用。

- **缩容（del-nodes）**时，先要使用 ` redis-cli --cluster reshard`将删除的机器上的 slots转移到其他机器上，然后使用`redis-cli --cluster del-nodes`将机器从集群中删除。

#### 10.5 集群模式的优缺点

##### 优点

- 可扩展性：支持在线扩容，节点可动态添加或删除;
- 基于**主从模式**，拥有主从模式的所有优点，如支持主从复制，能自动进行主从同步；可以进行读写分离。
- 每个 Master 的数据都不一样，内存的可用性较高，支持海量数据的存储，缓解了高并发写的问题。

##### 缺点

- Redis Cluster 是无中心节点的集群架构，依靠 Goss 协议(谣言传播)协同自动化修复集群的状态。但 GossIp 有消息延时和消息冗余的问题，在集群节点数量过多的时候，节点之间需要不断进行 PING/PANG 通讯，不必要的流量占用了大量的网络资源。虽然 Reds4.0 对此进行了优化，但这个问题仍然存在。
- 数据迁移问题：Redis Cluster 可以进行节点的动态扩容缩容，这一过程，在目前实现中，还处于半自动状态，需要人工介入。在扩缩容的时候，需要人为进行数据迁移。



### 11. 如何实现Redis的高可用？

> 所谓的高可用，也叫 HA（High Availability），是分布式系统架构设计中必须考虑的因素之一，它通常是指，通过设计减少系统不能提供服务的时间。如果在实际生产中，如果 redis 只部署一个节点，当机器故障时，整改服务都不能提供服务了。这就是我们常说的单点故障。如果 redis 部署了多台，当一台或几台故障时，整个系统依然可以对外提供服务，这样就提高了服务的可用性。

redis 高可用的两种模式：**哨兵模式**，**集群模式**。（把上面9和10详细说一下）
