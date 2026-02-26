Redis

Remote Dictionary Server

现在最受欢迎的 NoSQL 数据库之一，Redis 是一个使用 ANSI C编写的开源、包含多个数据结构、支持网络、基于内存、可选持久性的键值对存储数据

特性：

- 基于内存运行，性能高效（每秒可以处理超过 10 万次读写操作）
- 支持分布式，理论上可以无限扩展
- key-value 存储系统（key 是字符串，键有字符串、列表、集合、散列表、有序集合等）
- 开源的使用 ANSI C 语言编写、遵守 BSD 协议、支持网络、可基于内存亦可持久化的日志类、key-value 数据库，并提供多种语言的 API

## 数据类型

主要有 5 种数据类型，包括 String、List、Set、ZSet、Hash，

高级数据结构 bitmap，HyperLogLog 底层是基于String，Geo 基于 ZSet

### string

最基础的数据结构，首先键都是字符串类型，其他几种数据结构都是在字符串类型基础上构建的

数据结构为简单动态字符串（Simple Dynamic String，缩写 SDS）

采用预分配冗余空间的方式来减少内存的频繁分配

长度小于 1M 时，扩容都是加倍现有空间，

超过 1M 时，扩容时一次只会多扩 1M 的空间

值可以是字符串（JSON、XML）、数字（整数、浮点数）、二进制（图片、音频、视频）

最大长度不能超过 512MB

#### encoding

##### int

长度小于20（2^64，为 20 位），且为长整型数字

##### embstr 

长度小于等于44，只分配一次内存空间，

##### raw

 长度大于44，append 会修改字符串底层的类型

#### 命令

get、set： 获取、设置值

getset ： 将给定 key 的值设为 value ，并返回 key 的旧值，原值不为字符串类型，返回错误

incr、decr： 自增、自减，必须是整数，浮点数不行

#### 使用场景

##### 缓存功能

Redis 作为缓存层，MySQL作为存储层，绝大部分请求的数据都是从 Redis 中获取，由于 Redis 具有支撑高并发的特性，所以缓存通常能寄到加速读写和降低后端压力的作用

###### 计数

使用 Redis 作为计数的基础工具，可以实现快速计数、查询缓存的功能，同时数据可以异步落地到其它数据源

###### 共享 Session

###### 限速

短信接口限制访问次数、一个 IP 地址访问次数

### list

链表

存储多个有序的字符串，列表中的每个字符串称为元素（element）

命令： lpush、lpop

列表类型有两个特点： 

列表中的元素是有序的，可以通过索引下标获取某个元素或者某个范围内的元素列表

列表中的元素是可以重复的

数据结构为快速链表 quickList

首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构就是 ziplist，压缩列表

ziplist 是由一系列特殊编码的连续内存块组成的顺序结构，类似与数组，在内存中是连续存储的，

不同于数组，每个元素所占的内存大小可以不同

不存储 prev 和 next 指针，而是存储上一个节点长度和当前节点长度，牺牲部分读写性能，换取高效的内存空间利用率，节省内存 

![在这里插入图片描述](https://img-blog.csdnimg.cn/b39e06a28d00467c89cfc2b9b332a324.png#pic_center)

- zlbytes， 记录压缩列表所占用的内存字节数，在对压缩列表中内存重分配，或者计算 zlend 的位置时使用
- zltail， 记录了尾结点至起始节点的偏移量，通过这个偏移量，可以快速确定最后一个 entry 节点的地址
- zllen，记录了 entry 节点的数量，当 zllen 的值小于 63535，这个值就是节点数量。大于65535，节点的真实数量需要遍历整个压缩列表才能得出 
- entry，压缩列表中所包含的每个节点。每个节点根据该节点的内容来决定
- zlend，特殊值0xFF，标记压缩列表的末端，表示该压缩列表到此为止

​	entry 节点的结构如下

![在这里插入图片描述](https://img-blog.csdnimg.cn/9707dfc4336e474f8862658a4efd0b5e.png#pic_center)

- prev_entry_len，记录前驱节点的长度
- encoding，记录当前节点的 value 成员的数据类型以及长度
- value，根据 encoding 来保存字节数组或整数

所有的元素分配的是一块连续的内存

数据量比较多的时候才会改成 quicklist

因为普通的链表需要的附加指针空间太大，会比较浪费空间，需要额外的指针 prev 和 next

![image-20220325161041093](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220325161041093.png)

将链表和 ziplist 结合组成了 quicklist，即多个 ziplist 使用双向指针串起来使用。既满足了快速的插入删除性能，又不会出现太大的空间冗余

#### 使用场景

##### 消息队列

Redis 的 lpush + brpop 命令组合即可实现阻塞队列，生产者客户端使用 lrpush 从列表左侧插入元素，多个消费者客户端使用 brpop 命令阻塞式的"抢"列表尾部的元素，多个客户端保证了消费的负载均衡和高可用性

##### 文章列表

有序的，同时支持按照索引范围获取元素

##### 实现其他数据结构

lpush + lpop = Stack （栈）

lpush + rpop = Queue （队列）

### hash

数组 + 链表

键值对集合

#### 数据结构 

ziplist  长度较短且个数较少，否则使用 hashtable

#### 使用场景

比较适宜存放对象类型的数据

数据表记录

| id   | name | age  | sex  |
| ---- | ---- | ---- | ---- |
| 1    | king | 18   | boy  |

##### **使用 string 类型**

set user:1:name king;

set user:1:age 18;

set user:1:sex boy;

优点： 简单直观，每个键对应一个值

缺点： 键数过多，占用内存多，用户信息过于分散，不用于生产环境

##### **将对象序列化存入 redis** 

set user:1 serialize (userInfo);

优点： 编程简单，若使用序列化合理内存使用率高

缺点： 序列化与反序列化有一定开销，更新属性时需要把 userInfo 取出来反序列化，更新后再序列化到 redis

##### **使用 hash 类型**

hmset user:1 name king age 18 sex boy

优点：简单直观，使用合理可减少内存空间消耗

缺点： 要控制内部编码格式，不恰当的格式会消耗更多内存

#### 数据结构

field-value 长度较短且个数较少时，使用 ziplist，否则使用 hashtable

### set

无序集合，集合中的元素没有先后顺序，元素不重复，自动排重

底层是 hash 表，增删查的复杂度都是 O(1)

#### 使用场景

提供了判断某个成员是否在一个 set 集合内的重要接口 ，可以基于 set 轻易实现交集、并集、差集的操作

典型使用场景 标签 tag，

生成随机数进行比如抽奖活动、社交图谱等

### zset

有序、不能重复。和列表使用索引下标作为排序依据不同的是，给每个元素设置一个分数（score）作为排序的依据。有序集合的元素不能重复，但是 score 可以重复

提供了获取分数和元素范围查询、计算成员排名等功能

```
zadd <key><score1><value1><score2><value2>

zrange <key><start><stop>  [WITHSCORES]  带WITHSCORES，可以让分数一起和值返回到结果集

zrangebyscore key minmax [withscores] [limit offset count]
返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列

zrevrangebyscore key maxmin [withscores] [limit offset count]               
同上，改为从大到小排列

zincrby <key><increment><value>      为元素的score加上增量

zrem  <key><value>删除该集合下，指定值的元素 

zcount <key><min><max>统计该集合，分数区间内的元素个数 

zrank <key><value>返回该值在集合中的排名，从0开始
```

#### 数据结构

hash，hash 的作用就是关联元素 value 和权重 score，保障元素 value 的唯一性，可以通过元素 value 找到相应的 score 值

跳跃表，目的在于给元素 value 排序，根据 score 的范围获取元素列表

平衡树或红黑树虽然效率高，但结构复杂，链表查询需要遍历所有效率低

跳表效率堪比红黑树，实现远比红黑树简单

在原有的有序链表上面增加了多级索引，通过索引来实现快速查找

![image-20220514214054113](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220514214054113.png)



#### 使用场景

排行榜系统

榜单维度：时间、播放数量、获得的赞数

### bitmap

本身不是一种数据结构，实际上它就是字符串，但是它可以对字符串的位进行操作

单独提供了一套命令，类似一个以位为单位的数组，数组的每个单元只能存储0 和 1，数组的下标在 Bitmaps 中叫做偏移量

#### 使用场景

适合需要保存状态信息（是否签到、是否登录）并需要进一步对这些信息进行分析的场景

用户签到情况、用户行为统计（是否点赞过某个视频）

布隆过滤器

## redis 为什么那么快

- 高效的存储介质

- 优良的底层数据结构

  O（1）

- 高效的网络 IO 模型

- 高效的线程模型

单线程 + IO 多路复用技术

多路复用是指使用一个线程来检查多个文件描述符（Socket）的就绪状态，比如调用 select 和 poll 函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时，得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动单线程执行（比如使用线程池）

## 数据库设计

hash（key）取模计算得到地址指针，指向数据存储地址

key 值哈希冲突：链表地址，头插法（尾插法）

## 配置文件

tcp-backlog，连接队列，backlog 队列总和 = 未完成三次握手队列 + 已经完成三次握手队列

高并发环境下需要一个高 backlog 值来避免慢客户端连接问题

## 事务

事务是一个单独的隔离操作，事务中的所有命令都会序列化、按顺序地执行。

事务在执行的过程中，不会被其他客户端发送来的命令请求所打断

串联多个命令防止别的命令插队

### Multi、Exec、Discard

输入 Multi 命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入 Exec 后， Redis 会将之前的命令队列中的命令依次执行

watch 

multi

### 事务特性

#### 单独的隔离操作

事务中的所有命令都会序列化、按顺序执行。事务在执行的过程中，不会被其他客户端发送来的命令请求打断

#### 没有隔离级别的概念

队列中的命令没有提交之前都不会实际执行

#### 不保证原子性

事务中有一条命令执行过程中失败，其后的命令仍然会被执行，没有回滚

## 持久化 

### RDB

Fork 

复制一个与当前进程一样的子进程，新进程的所有数据（变量、环境变量、程序计数器等）都和原进程一致

写时复制

一般情况父进程和子进程会共用同一段物理内存，只有进程空间的内存发生变化时，才会将父进程的内存复制一份给子进程

在redis.conf中配置文件名称，默认为dump.rdb

文件位置,默认启动命令所在目录  dir ./

![image-20220326131450995](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220326131450995.png)



#### 缺点 

最后一次持久化的数据可能丢失

内存中数据量非常大的场景线下，主进程 fork 子进程花费的时间较多，此期间 redis 会暂停对外提供服务

### AOF

官方推荐都启用

如果对数据不敏感，可以单独使用 RDB

不建议单独用 AOF，因为可能会出现 Bug

如果只是做纯内存缓存，可以都不用

可以做到每隔一秒备份一次数据，后台进程执行 fsync 操作

#### 主要缺点

可能导致文件体积过大，当系统重启恢复数据时加载数据会非常慢，几十 G 的数据可能需要几小时才能加载完，这个耗时并不是因为磁盘文件读取速度慢，而是由于读取的所有命令都要在内存中执行一遍

使用 AOF 的方式，redis 读写性能会有所下降



## 高可用方案

###  主从复制 

1.  从节点执行 **slaveof** masterIp port，保存主节点信息
2.  从节点中的定时任务发现主节点信息，建立和主节点的 socket 连接
3.  从节点发送信号，主节点返回，两边能互相通信
4.  连接建立后，主节点将所有数据发送给从节点（数据同步）
5.  主节点把当前的数据同步给从节点后，便完成了复制过程。接下来，主节点就会持续的把写命令发送给从节点，保证主从数据一致性

runId： 每个 redis 节点启动后都会生成唯一的 uuid，每次 redis 重启后，runId 都会发生变化

offset： 主从节点各自维护自己的复制偏移量 offset，当主节点有写入命令时，offset=offset + 命令的字节长度。从节点在收到主节点发送的命令后，也会增加自己的 offset，并把自己的 offset 发送给主节点。主节点同时保存自己的 offset 和从节点的 offset， 通过对比 offset 来判断主从节点数据是否一致

repl_backlog_size： 保存在主节点上的一个固定长度的先进先出队列，默认大小是 1MB

全量复制：（同步的是RDB数据）

- 从节点发送 psync 命令，psync runId offset （由于是第一次， runid 为 ？， offset 为 -1）
- 主节点返回 FULLRESYNC runid offset， runId 是主节点的 runId，offset 是主节点目前的 offset。 从节点保存信息
- 主节点启动 bgsave 命令 fork 子进程进行 RDB 持久化
- 主节点将 RDB 文件发送给从节点，到从节点加载数据完成前，写命令写入缓存区 replication buffer
- 从节点清理本地数据并加载 RDB，如果开启了 AOF 会重写 AOF，主库会把 replication buffer 中的修改操作发给从库执行

增量复制：（ 同步的是写命令）

1. 复制偏移量： psync runid offset 
2. 复制积压缓冲区： 当主从节点 offset 的差距过大超过缓冲区长度时，将无法执行部分复制，只能执行全量复制
   - 如果从节点保存的 runid 与主节点现在的 runid 相同，说明主从节点之前同步过，主节点会继续尝试使用部分复制（能不能部分复制还要看 offset 和复制积压缓冲区的情况）
   - 如果从节点保存的 runid 与主节点现在的 runid 不同，说明从节点在断线前同步 Redis 节点并不是当前的主节点，只能进行全量复制

![image-20220313154849833](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220313154849833.png) 

### 哨兵模式

sentinel，哨兵是 redis 集群中非常重要的一个组件，主要有以下功能：

- 集群监控： 负责监控 redis master 和 slave 进程是否正常工作
- 消息通知： 如果某个 redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员
- 故障转移： 如果 master node 挂掉了，会自动转移到 slave node 上
- 配置中心： 如果故障转移发生了，通知 client 客户端新的 master 地址

哨兵用于实现 redis 集群的高可用，本身也是分布式的，作为一个哨兵集群去运行，互相协同工作

- 故障转移时，判断一个 master node 是否宕机了，需要大部分的哨兵都同意才行，涉及到了分布式选举
- 即使部分哨兵节点挂掉了，哨兵集群还是能正常工作的
- 哨兵通常需要 3 个实例，来保证自己的健壮性
- 哨兵 + redis 主从的部署架构，不能 保证数据零丢失的，只能保证 redis 集群的高可用性
- 对于哨兵 + redis 主从这种复杂的部署架构，尽量在测试环境和生产环境，都进行充足的测试和演练

选举条件

- redis.conf 配置文件 replica-priority | slave-priority，值越小优先级越高
- 偏移量最大，原主机数据最全的
- runid最小的服务



### Redis Sharding

客户端 Sharding ，Redis Cluster 出来前，业界普遍使用的多 Redis 实例集群方法，由客户端决定分片，其主要思想是采用哈希算法将 Redis 数据的 key 进行散列，通过 hash 函数，特定的 key 会映射到特定的 Redis 节点上

**优点**

简单，服务端的 Redis 实例彼此独立，相互无关联，每个 Redis 实例像单服务器一样运行，非常容易线性扩展，灵活性很强

**缺点**

由于 sharding 处理放到客户端，规模进一步扩大时给运维带来挑战

不支持动态增删节点。服务端 Redis 实例群拓扑结构有变化时，每个客户端都需要更新调整。连接不能共享，当应用规模增大时，资源浪费制约优化

### Redis Cluster 

服务端分片，允许单点故障，3.0 版本开始正式提供。采用 slot 的概念，一共分成 16384 (0~16384)个槽。将请求发送到任意节点，接收到请求的节点会将查询请求发送到正确的节点上执行 

方案说明

- 通过哈希的方式，将数据分片，每个节点均分存储一定哈希槽（哈希值）区间的数据，默认分配了 16384 个槽位
- 每份数据分片会存储在多个互为主从的多节点上
- 数据先写入主节点，再同步到从节点（支持配置为阻塞同步）
- 同一分片多个节点间的数据不保持强一致性
- 读取数据时，当客户端操作的 key 没有分配在该节点上时，redis 会返回转向指令，指向正确的节点
- 扩容时需要把旧节点的数据迁移一部分到新节点

某段插槽的主从都挂掉

- redis.conf 中 cluster-require-full-coverage 为 yes，整个集群都挂掉
- redis.conf 中 cluster-require-full-coverage 为 no，该插槽数据全都不能使用

Redis Cluster 架构下，每个 redis 要放开两个两个端口号，比如 6379、16379

16379 端口号是用来进行节点通信的，也就是 cluster bus 的通信，用来进行故障检查、配置更新、故障转移授权。cluster bus 用了另外一种二进制的协议，gossip 协议。用于节点间进行高效的数据转换，占用更少的网络带宽和处理时间

**优点**

- 无中心架构，支持动态扩容，对业务透明
- 具备 Sentinel 的监控和自动 Failover（故障转移）能力
- 客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可
- 高性能，客户端直连 redis 服务，免去了 proxy 代理的损耗

**缺点**

- 不支持多键操作，多键事务，lua 脚本

- 只能使用 0 号数据库

  ```
  配置 redis.conf
  include /home/bigdata/redis.conf
  port 6379
  pidfile "/var/run/redis_6379.pid"
  dbfilename "dump6379.rdb"
  dir "/home/bigdata/redis_cluster"
  logfile "/home/bigdata/redis_cluster/redis_err_6379.log"
  cluster-enablinclueed yes
  cluster-config-file nodes-6379.conf
  cluster-node-timeout 15000
      
      
  redis-cli --cluster create --cluster-replicas 1 192.168.231.129:6379 192.168.231.129:6380 192.168.231.129:6381 192.168.231.129:6389 192.168.231.129:6390 192.168.231.129:6391
  --cluster-replicas 表示为集群中的每个主节点创建一个从节点
  ```

  

## 简述Redis 主从同步机制

1.  从节点执行 **slaveof** masterIp port，保存主节点信息
2.  从节点中的定时任务发现主节点信息，建立和主节点的 socket 连接
3.  从节点发送信号，主节点返回，两边能互相通信
4.  连接建立后，主节点将所有数据发送给从节点（数据同步）
5.  主节点把当前的数据同步给从节点后，便完成了复制过程。接下来，主节点就会持续的把写命令发送给从节点，保证主从数据一致性

runId： 每个 redis 节点启动后都会生成唯一的 uuid，每次 redis 重启后，runId 都会发生变化

offset： 主从节点各自维护自己的复制偏移量 offset，当主节点有写入命令时，offset=offset + 命令的字节长度。从节点在收到主节点发送的命令后，也会增加自己的 offset，并把自己的 offset 发送给主节点。主节点同时保存自己的 offset 和从节点的 offset， 通过对比 offset 来判断主从节点数据是否一致

主从复制使用 RDB

- RDB 文件是二进制文件，网络传输 RDB 和写入磁盘的 IO 效率都比 AOF 高
- 从库进行数据恢复的时候，RDB 的恢复效率也要高于 AOF

基于长连接的命令传播

主从库完成了全量复制，它们之间会一直维护一个网络连接，主库通过这个连接将收到的命令操作发给从库，并维持心跳机制

- PING，主节点向从节点发送 PING 命令
- REPLCONF ACK <replication_offset>，从节点每秒一次向主节点发送当前的复制偏移量

### replication buffer

全量复制时的 buffer，对应于每个 slave

通过 config set client-output-buffer-limit slave 设置

```
config set client-output-buffer-limit "slave 536870912 536870912 0"
```

推荐设置为 512M

值太小，会导致主从复制连接断开，更严重的是导致主从上出现重新执行 bgsave 和 rdb 重传操作无限循环

值太大，或网络延迟较大时，可能导致该缓冲区大小超过限制，断开连接

可能引起循环：全量复制 -> replication buffer 溢出导致连接中断 ->  重连 -> 全量复制 

### repl_backlog_buffer 

增量复制，所有 slave 公用

通过 repl_backlog_size 设置大小，默认大小是 1MB，产线一般配置为 128M

一般计算公式

```
repl_backlog_buffer = 2 * econd * write_size_per_second

second: 从服务器断开重连主服务器所需平均时间
write_size_per_second: master 平均每秒产生的数据量大小 （写命令和数据总和）
```

断开重连后，将中断期间主节点执行的写命令发送给从节点，与全量复制相比更加高效

全量复制：（同步的是RDB数据）

- 从节点发送 psync 命令，psync runid offset （由于是第一次， runid 为 ？， offset 为 -1）
- 主节点返回 FULLRESYNC runid offset， runId 是主节点的 runId，offset 是主节点目前的 offset。 从节点保存信息
- 主节点启动 bgsave 命令 fork 子进程进行 RDB持久化
- 主节点将 RDB 文件发送给从节点，到从节点加载数据完成前，写命令写入缓存区
- 从节点清理本地数据并加载 RDB，如果开启了 AOF 会重写 AOF

增量复制：（同步的是写命令）

1. 复制偏移量： psync runid offset 
2. 复制积压缓冲区： 当主从节点 offset 的差距过大超过缓冲区长度时，将无法执行部分复制，只能执行全量复制
   - 如果从节点保存的 runid 与主节点现在的 runid 相同，说明主从节点之前同步过，主节点会继续尝试使用部分复制（能不能部分复制还要看 offset 和复制积压缓冲区的情况）
   - 如果从节点保存的 runid 与主节点现在的 runid 不同，说明从节点在断线前同步 Redis 节点并不是当前的主节点，只能进行全量复制

## Redis Rehash

https://tech.meituan.com/2018/07/27/redis-rehash-practice-optimization.html

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/4e1551b0.png)



一个 RedisDB 对应一个 Dict

一个 Dict 对应 2 个 Dictht，正常情况下只用到 ht[0]；ht[1] 在 Rehash 时使用

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/1ff650e3.png)

### 渐进式 rehash

元素数量过大时，rehash 将会是一个非常耗时的操作，庞大的计算量可能会导致服务器在一段时间内停止服务。所以 rehash 这个动作不能一次性、集中式的完成，而是分多次、渐进式地完成

### 扩容条件

1. 没有在执行 BGSAVE 或 BGREWRITEAOF 命令，并且哈希表的负载因子（hash 冲突）大于等于1
2. 目前在执行 BGSAVE 或 BGREWRITEAOF 命令，并且哈希表的负载因子大于等于5

### 缩容的条件

哈希表的负载因子小于 0.1  

BGSAVE 或 BGREWRITEAOF 命令是否在执行，Redis 服务器哈希表执行扩容所需的负载因子不同。因为此时子进程使用写时复制Copy On Write，需要占用一定的内存，所以需要提高扩容所需的负载因子，从而尽可能避免在子进程存在期间进行扩容，可以避免不必要的内存写入操作，最大限度节约内存                                                                                           



## Scan



## 简述 redis 分布式锁实现

setnx + setex： 存在设置超时时间失败的情况，导致死锁

set（key, value, nx, px）: 将 setnx + setex 变成原子操作

问题：

- 任务超时，锁自动释放，导致并发问题。使用 redisson 解决（看门狗监听，自动续期）
- 加锁和释放锁不是一个线程的问题。在 value 中放入 uuid （线程唯一标识），删除锁时判断该标识（使用 lua 保证原子操作）（执行超时，锁自动释放，另一线程加锁后，原线程又去释放锁）
- 不可重入，使用 redisson 解决（实现机制类似 AQS，计数）
- 异步复制可能造成锁丢失，使用 redLock 解决

1. 顺序向五个节点请求加锁
2. 根据一定的超时时间来判断是不是跳过该节点
3. 三个节点加锁成功并且花费时间小于锁的有效期
4. 认定加锁成功

## 缓存穿透

 缓存穿透是指缓存和数据库中都没有的数据，导致所有的请求都落到数据库上，造成数据库短时间内承受大量请求而宕机

解决方案：

- 接口层增加校验，如用户鉴权校验，id 做基础校验，id<=0 的直接拦截
- 从缓存取不到的数据，在数据库中也没有取到，这时也可以将 key-value 对写为 key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也无法使用）。这样可以防止攻击用户仿佛用同一个 id 暴力攻击
- 采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对数据库的查询压力           

## 缓存击穿

缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），大量并发用户同时读缓存没有读到数据，又同时去数据库取数据，引起数据库压力瞬间增大，造成过大压力。和缓存雪崩不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了     

解决方案：

- 设置热点数据永远不过期
- 使用 mutex

缓存失效时（判断值为空），不是立即查询数据库，先执行 SETNX 上锁后，再查询数据库并回设缓存，否则重试查询缓存的方法

```c#
public string Get(string key){
	string value = redis.Set(key);
    if(value == null){
        if(redis.Setnx(mutex_key,3*60) == 1){
            value = db.Get(key);
            redis.Set(key, value, expire_secs);
            redis.Del(mutex_key);
        }
        else{
            Sleep(50);
            Get(key);
        }
	}	
}
```





## 缓存雪崩

缓存雪崩是指缓存同一时间大面积的失效，后面的请求都会落到数据库上，造成数据库短时间内承受大量请求而崩掉

解决方案：

- 缓存预热 
- 缓存数据设置随机过期时间，防止同一时间大量数据过期现象发生。
- 给每一个缓存数据增加相应的缓存标记，记录缓存是否失效，如果缓存标记失效，则更新数据缓存

## 分布式锁

分布式系统多线程、多进程部署在不同的机器上，单机的并发控制策略失效

分布式锁：控制共享资源访问的互斥机制

方案

1. 基于数据库实现分布式锁
2. 基于缓存（redis 等），性能最高

 为了确保分布式锁可用，必须满足以下条件：

- 互斥性
- 不会发生死锁
- 加锁和解锁必须是同一个客户端
- 加锁和解锁必须具有原子性

## Redis6.0 新功能

### ACL

Access Control List，访问控制列表

允许根据可以执行的命令和可以访问的键来限制某些连接

Redis 5 前，安全规则只有密码控制，以及通过 rename 来调整高危命令 （flushdb， keys *， shutdown）

对用户进行更细粒度的权限控制

- 接入权限：用户名和密码
- 可以执行的命令
- 可以操作的 key  

act list 展现用户权限列表

![image-20220326154101429](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220326154101429.png)

acl cat  查看添加权限指令列表

![image-20220326154126715](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220326154126715.png)、

加参数类型名可以查看类型下具体命令

![image-20220326154148400](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220326154148400.png)

acl whoami  查看当前用户

aclsetuser 创建和编辑用户ACL

### IO 多线程

客户端交互部分的网络 IO 交互处理模块多线程，多线程用来处理网络数据的读写和协议解析

执行命令依然是单线程

配置文件，默认不开启

io-threads-do-read no

io-threads 4 

### 工具支持 Cluster

 旧版需要单独安装 ruby 环境
