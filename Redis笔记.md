# Redis笔记

> 概述

Redis(Remote Dictionary Server)。即远程字典服务，是一个开源的，使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，提供多语言的API。



> Redis 能干嘛？

1.内存储存，持久化，AOF,RDB

2.效率高，可用于高速缓存

3.发布订阅系统

4.计时器，计数器(浏览量)

...

> 特性

1.多样的数据类型

2.持久化

3.集群

4.事物

...

> 基础知识

1.Redis默认有16个数据库，默认使用的是第0个数据库，可以使用select切换数据库

```shell
select 1 # 切换数据库
dbsize # 查看db大小
keys * # 查看所有key
flushall # 清空数据库
exists key # 判断当前key是否存在
move key 1 # 移除当前的key
expire key second # 设置过期时间
ttl key # 查看key剩余过期时间
type key # 查看key的类型
```



> Redis Linux 安装

```shell
# 1.下载 https://redis.io
# 2.解压 
cd /opt
tar -zxvf redis-xxxxx.tar.gz
# 3.编译
make
# 4.编译安装
make install
# redis 默认安装路径 usr/local/bin
```



> Redis五大数据类型

> 1、String

```shell
set key value # 设置key-value
get key # 获取key的value
exists key # 判断key是否存在
append key value # 追加字符串，若key不存在相当于set key value
strlen key # 获取字符串的长度
incr key # 当前key的value +1
decr key # 当前key的value -1
incrby key 10 # 当前key + 10
decrby key 10 # 当前key - 10
getrange key 0 3 # 字符串范围 (getrange key 0 -1 获取全部字符串)
setrange key 1 xxx # 替换指定位置开始的字符串
setex key second value # (set with expire) 设置过期时间
setnx key value # (set if not with exists) 不存在再设置(分布式锁中使用)
mset k1 v1 k2 v2 # 批量设置
mget k1 k2 # 批量获取
msetnx k1 v1 k2 v2 # 不存在再设置(批量 原子性操作 一起成功 一起失败)
getset key value # 先获取取原值再设置新址
```



> 2 、List

redis中，可以将list用作栈，队列，阻塞队列的数据结构

```shell
lpush key v1 v2 .... # 将一个值或者多个值插入到列表的头部(左)
rpush key v1 v2 .... # 将一个值或者多个值插入到列表的尾部(右)
lrange key start end # 用过区间获取具体的值(0 -1 区间获取全部值)
lpop key # 移除列表头部的第一个值(左)
rpop key # 移除列表尾部的第一个值(右)
lindex key index # 通过索引获取值
llen key # 获取列表长度
lrem key count value # 移除list集合中指定个数的value 精确匹配
ltrim key start stop # 通过下标截取指定长度，list已经改变，只剩下截取之后的元素
rpoplpush key otherkey # 移除列表中最后一个元素,并将它插入到另一个列表的头部
lset key index value # 将列表中指定下标的值替换为另外一个值，更新操作(如果列表不存在 报错)
linsert key before v1 v2 # 在v1前插入v2
linsert key after v1 v2 # 在v1后插入v2
```

- 实际上是一个链表，before after left right 都可以插入值
- 如果key不存在，创建新的链表
- 如果key存在，新增内容
- 如果移除了所有值，空链表，也代表不存在
- 在两边插入或改动值，效率最高，中间元素，相对低一些
- 消息排队，消息队列，栈



> 3、Set

set中的值不能重复

```shell
sadd key value # 添加元素
smembers key # 查看指定set中所有元素
sismember key value # 判断某一个值在指定set中是否存在
scard key # 获取set中的内容元素个数
srem key value # 移除set中指定的元素
srandmember key count # 随机选出指定个数的成员
spop key # 随机移除元素
smove oldkey newkey member # 将一个指定的值，从一个set移动到另一个set
sdiff key.... # 获取多个set差集
sinter key... # 获取多个set交集
sunion key... # 获取多个set并集
```



> 4、Hash

Map集合 Key-(Key-Value)

```shell
hset key field value # 存入一个具体键值对
hget key field # 获取一个字段值
hmset key field value field1 value1 field2 value2.... # 存入多个具体键值对
hmget key field field1 field2 # 获取多个字段值
hgetall key # 获取全部数据
hdel key field # 删除hash指定的key字段，对应value也就没有了
hlen key # 获取hash中字段数量
hexists key field # 判断hash中某个字段是否存在
hkeys key # 获取hash中全部key
hvals key # 获取hash张全部的value
hincrby key field 1 # hash中指定key的value加1
hdecrby key field 1 # hash中指定key的value减1
hsetnx key field value # 如果hash中指定的key不存在则创建，存在则创建失败
```

hash应用场景：变更的数据，比如用户信息，以及其他经常变动的信息；hash更加适合对象的存储，String更适合字符串的存储



> 5、Zset

有序集合，在set的基础上增加了一个排序的值

```shell
zadd key score value # 添加元素
zrange key 0 1 # 通过索引区间返回有序集合指定区间内的成员 (0 -1 返回全部)
zrangebyscore key min max # 排序斌返回 从小到大  例如 zrangebyscore key1 -inf +inf
zrevrange key 0 -1 # 排序并返回 从大到小
zrem key value # 移除指定元素
zcrad key # 获取有序集合中的数量
zcount key start stop # 获取指定区间中的成员数量
```



> Redis 三种特殊数据类型

1. geospatial

``` shell
#geoadd 添加地理位置

#geopop 获取指定成员的经度和纬度

#geodist 查看成员间的的直线距离

#georadius 以给定经纬度为中心，找出某一半径内的元素

#georadiusbymember 以给定成员为中心，找出某一半径内的元素

#geohash 返回一个或多个位置元素的geohash表示  将二维的经纬度转换成一维的11位字符串 如果两个字符串越接近，则距离越近。
```



1. hyperloglog

基数 （不重复的元素个数） 可以接受误差 大概有0.81%的错误率

Redis hyperloglog 基数统计算法：
优点： 占用内存固定，存放2^64不同的元素的技术，只需要占用12KB内存
**网页的UV （一个人访问一个网站多次，统计出还是一个人）**
传统的方式：set集合保存用户id，统计set中用户数量。 但是相对消耗更多内存，我们的目的并不是保存用户id，目的只是计数。

```shell
PFadd key element # 创建一组元素
PFcount key # 统计元素基数
PFmerge key3 key1 key2 # 合并两组key1 key2 => key3 并集
```



1. bitmap

位存储

统计用户信息 活跃 不活跃 登录 未登录 打卡
两个状态的 都可以使用bitmaps
bitmaps位图数据结构，都是操作二进制位来进行记录的，非0即1





> Redis 事务

Redis 单条命令时保证原子性的，但是事务不保证原子性

Redis事务的本质：一组命令的集合，一个事务中所有命令都会被序列化，在事务执行的过程中，会按照顺序执行。**一次性、顺序性、排他性**

Redis事务没有隔离级别的概念

所有命令在事务中，并没有直接执行，只有在发起执行命令的时候才会执行



开启事务 multi

命令入队 ...

执行事务 exec 、放弃事务 discard



**编译型异常（代码有问题，命令有错），事务中所有的命令都不会被执行**

**运行时异常 ，如果事务队列中存在语法性，那么执行命令的时候，其他命令是可以正常执行的，错误命令抛出异常**



> Redis 实现乐观锁

监控 watch、unwatch (解锁，如果事务执行失败，先解锁，然后再次手动取监视)

悲观锁：无论做什么都加锁

乐观锁：获取version，更新时比较version





### Redis.conf 详解

1、单位 units

2、包含includes 	可以包含多个配置文件，运行时合并成一个

3、网络 network

```shell
bind 127.0.0.1 # 绑定访问ip
protected-mode yes # 保护模式 yes 开始 no 关闭
port 6379 # 端口
```

4、通用 general

```shell
daemonize yes #以守护进程的方式进行,默认是no
pidfile /var/run/redis_6379.pid  #如果以后台（守护进程）的方式运行，我们就需要指定一个pid文件
```

5、日志

```shell
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)   （生产环境）
# warning (only very important / critical messages are logged)
loglevel notice  #日志级别
logfile ""     #日志文件名
databases 16   #数据库个数   默认16
always-show-logo yes   #是否总是展示logo
```

6、快照 snapshotting

持久化，在规定时间内，执行了多少次操作，则会持久化到文件 .rdb .aof
redis是内存数据库，如果没有持久化，断电数据丢失

```shell
save 900 1   #如果900秒内，至少一个key进行了修改，那么就进行持久化操作
save 300 10   #如果300秒内，至少有10个key进行了修改，那么就进行持久化操作
save 60 10000  #如果60秒内，至少有10000个key进行了修改，那么就进行持久化操作
stop-writes-on-bgsave-error yes  #持久化如果出错，是否还需要继续工作
rdbcompression yes  #是否压缩.rdb文件（会消耗一定的cpu资源）
rdbchecksum yes    #保存.rdb文件的时候，进行错误检查校验
dir ./     #.rdb文件保存路径
```

7、复制（replication 主从复制）

``` shell
replicaof 127.0.0.1 6379
```



8、安全 security

```shell
requirepass root    #设置密码为root  默认是没有密码的
```

9、客户端限制（clients）

```shell
maxclients 10000   #最大连接数
```

10、内存设置(memory management)

```shell
maxmemory <bytes>  #最大内存容量
maxmemory-policy noeviction   #内存处理策略  内存达到上限之后的处理策略
    1、volatile-lru：只对设置了过期时间的key进行LRU（默认值）
    2、allkeys-lru ： 删除lru算法的key
    3、volatile-random：随机删除即将过期key
    4、allkeys-random：随机删除
    5、volatile-ttl ： 删除即将过期的
    6、noeviction ： 永不过期，返回错误
```

11、aof配置（append only mode）

```shell
appendonly no   #aof模式默认不开启,默认是使用rdb方式持久化的，在大部分情况下,rdb完全够用
appendfilename "appendonly.aof"  #.aof文件名
# appendfsync always  #每次修改都会执行同步，消耗性能
appendfsync everysec  #每秒执行一次同步，但是可能会丢失这1s的数据
# appendfsync no      #不执行同步，这个时候操作系统自己同步数据，速度最快
####重写规则####
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb  #如果aof文件大于64mb，太大了，fork一个新的进程来讲我们的文件进行重写。
```



###  Redis的持久化

#### RDB（Redis DataBase）

在指定时间间隔将内存中的数据集体快照写入磁盘，也就是行话讲的snapshot快照，它恢复时是将快照文件直接读到内存中。

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入一个临时文件中，待持久化过程结束后，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何I/O操作的。这就确保了极高的性能。如果需要大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加高效。

优点：
1、适合大规模数据恢复。
2、对数据完整性要求不高。

缺点：
1、需要一定时间间隔进程操作，如果在时间间隔内redis宕机，最后一次持久化后的数据丢失。
2、fork子进程的时候，会占用一定的内存空间。

默认的持久化方式就是RDB方式，一般情况下不需要修改这个配置。默认保存的rdb文件为**dump.rdb**

触发规则：
1、save的规则满足的情况下，自动触发rdb规则
2、flushall命令执行后，自定触发rdb规则
3、退出redis时，自动触发rdb规则

恢复rdb文件：
1、只需要将rdb文件放在redis的启动目录就可以，redis启动的时候会自动检查dump.rdb文件，自动恢复rdb文件中的数据

```
127.0.0.1:6379> config get dir1) "dir"2) "/usr/local/bin"  #如果这个目录下存在rdb文件，启动redis就是自动恢复其中数据127.0.0.1:6379>
```

在主从复制中，rdb就是备用了！从机上面！

#### AOF（Append Only File）

*将所有执行过的写命令都记录下来，history，在恢复的时候将记录的命令全部执行一遍。*

以日志的形式来记录每个写操作，将Redis执行过的的所有指令记录下来（读操作不记录），只许追加文件不许改写文件，redis启动时，会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容，将写指令从前到后执行一次以完成数据的恢复工作。

aof默认是不开启的，需要手动开启（appendonly yes）。aof默认保存的是appendonly.aof文件。
重启，redis就可以生效了。

如果aof文件损坏或有错误，redis无法启动。我们需要修复aof文件。aof文件损坏修复:redis-check-aof —fix appendonly.aof

优点：

```
# appendfsync always  #每次修改都会执行同步，消耗性能appendfsync everysec  #每秒执行一次同步，但是可能会丢失这1s的数据# appendfsync no      #不执行同步，这个时候操作系统自己同步数据，速度最快
```

1、每次修改都会执行同步，消耗性能,文件数据完整性更好。
2、每秒执行一次同步，但是可能会丢失这1s的数据
3、不执行同步，这个时候操作系统自己同步数据，速度最快

缺点：
1、相对于数据文件来说，aof文件远远大于rdb文件，修复的速度也较慢。
2、aof运行效率也要比rdb慢，所以我们redis默认的配置就是rdb持久化。

**扩展：**
1、RDB 持久化方式能够在指定的时间间隔内对你的数据进行快照存储
2、AOF 持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis 协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。
**3、只做缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化**
4、同时开启两种持久化方式
在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。
RDB 的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢？作者建议不要，因为RDB更适合用于备份数据库（AOF在不断变化不好备份），快速重启，而且不会有AOF可能潜在的Bug，留着作为一个万一的手段。
5、性能建议
因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留 save 900 1 这条规则。
如果Enable AOF ，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了，代价一是带来了持续的IO，二是AOF rewrite 的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上，默认超过原大小100%大小重写可以改到适当的数值。
如果不Enable AOF ，仅靠 Master-Slave Repllcation 实现高可用性也可以，能省掉一大笔IO，也减少了rewrite时带来的系统波动。代价是如果Master/Slave 同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave 中的 RDB文件，载入较新的那个，微博就是这种架构。



### Redis发布订阅

Redis发布订阅（pub/sub）是一种**消息通信模式**：发送者（pub）发送信息，订阅者（sub）接收信息。

Redis客户端可以订阅任意数量的频道。

订阅/发布消息图：
第一个：消息发送者 第二个：频道 第三个：消息订阅者

#### 命令

|                  命令                  |                描述                |
| :------------------------------------: | :--------------------------------: |
|     PSUBSCRIBE pattern [pattern..]     | 订阅一个或多个符合给定模式的频道。 |
|    PUNSUBSCRIBE pattern [pattern..]    | 退订一个或多个符合给定模式的频道。 |
| PUBSUB subcommand [argument[argument]] |      查看订阅与发布系统状态。      |
|        PUBLISH channel message         |         向指定频道发布消息         |
|     SUBSCRIBE channel [channel..]      |     订阅给定的一个或多个频道。     |
|     SUBSCRIBE channel [channel..]      |         退订一个或多个频道         |

**缺点**
如果一个客户端订阅了频道，但自己读取消息的速度却不够快的话，那么不断积压的消息会使redis输出缓冲区的体积变得越来越大，这可能使得redis本身的速度变慢，甚至直接崩溃。
这和数据传输可靠性有关，如果在订阅方断线，那么他将会丢失所有在短线期间发布者发布的消息。

**应用**
消息订阅：公众号订阅，微博关注等等（起始更多是使用消息队列来进行实现）
多人在线聊天室。

稍微复杂的场景，我们就会使用消息中间件MQ处理



### Redis主从复制

#### 概念

主从复制，是指将一台Redis服务器的数据，复制到其他Redis的服务器。前者称为主节点（master/leader），后者称为从节点（slave/follower）；数据的复制是单向的，只能从主节点到从节点。Master以写为主，Slave以读为主。

主从复制，读写分离！80%的情况下都是进行读的操作！减缓服务器压力！

**默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点（或没有从节点），但是一个从节点有且只有一个主节点。**

**主从复制的作用主要包括**

- 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
- 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
- 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是写少读多的场景下，通过多个节点分担负载，可以大大提高Redis服务器的并发量。
- 高可用（集群）基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

一般来说呀，要将redis运用于工程项目之中，只使用一台Redis是万万不能的（宕机），原因如下：
1、从结构上，单个redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载呀，压力较大。
2、从容量上，单个redis服务器内存容量有限，就算一台redis服务器内存容量为256GB，也不能将所有内存用作Redis存储内存，**一般来说，单台Redis最大使用内存不应该超过20GB。**

#### 环境配置

只需要配置从库，不需要配置主库，因为默认情况下，每台Redis服务器都是主节点。

修改3个配置文件：
1、端口
2、pid
3、log文件名字
4、rdb备份文件名字

#### 一主二从

默认情况下，每台redis服务器都是主节点；我们只需要配置从节点就好了（认老大）。

命令配置 (临时)

```shell
SLAVEOF 127.0.0.1 6379   #设置为主节点的从节点（认老大）
```

文件配置 (永久)

```shell
replicaof 127.0.0.1 6379
```

**注意：
1、主节点宕机，从节点依旧以从节点的角色连接主节点，没有写的权限；当主节点重新连接，从节点依旧可以直接获取到主节点写的信息！
2、如果是用命令行配置的从节点，从节点重启，主从配置失效！但是当重新配置成为从节点，立刻就会从主节点中获取值！
3、如果是配置文件配置的从节点，从节点宕机，主节点进行写操作，当从节点重新连接时，会从主节点中获取值，之前主节点写操作的信息依旧在此从节点存在！**



### 哨兵模式

（自动选举老大的模式）





### Redis的缓存穿透与雪崩

#### 缓存穿透

缓存穿透的概念很简单，用户想要查询一个数据，发现redis内存数据库没有，也就是缓存没有命中，于是向持久层数据库查询。发现也没有，于是本次查询失败。当用户很多的时候，缓存都没有命中，于是都去请求了持久层数据库。这会给持久层数据库造成很大的压力，这时候就出现了缓存穿透。

> 解决方案

**1、布隆过滤器**
布隆过滤器是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先校验，不符合则丢弃，从而避免了对底层存储系统的查询压力；

**2、缓存空对象**
当存储层不命中后，即使返回的空对象也将其缓存起来，同时设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源；

但是这种方法存在两个问题：

- 如果空值能被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键；
- 即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对需要保持数据一致性的业务会有影响。

#### 缓存击穿

这里需要注意和缓存穿透的区别。缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在屏障上凿开了一个洞。

当某个key在**过期的瞬间**，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并且回写缓存，会导致数据库压力瞬间变大。

>  解决方案

**1、设置热点数据不过期**
从缓存层面来看，没有设置过期时间，所以不会出现热点key过期后产生的问题。

**2、加互斥锁**
分布式锁：使用分布式锁，保证对于每个key同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁考验很大。

#### 缓存雪崩

缓存雪崩，是指在某一时间段，缓存集中过期失效。Redis宕机

产生雪崩的原因之一，比如零点抢购，商品时间集中放入缓存，假设缓存一小时。那么1点的时候，大量缓存集体过期，对于这批商品的访问查询，都落到了数据库。对于数据库而言，就会产生周期性的压力波峰，于是所有请求都会到达存储层，存储层调用会暴增，造成存储层也会挂掉的情况

> 解决方案

**1、Redis高可用**
这个思想的含义是，既然Redis也有可能挂掉，那多增加几台redis服务器，这样一台挂了还有其他的可以继续工作，其实就是搭建集群。

**2、限流降级**
这个解决方案的思想是，在缓存失效后，通过加锁或者队列来控制读取数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。

**3、数据预热**
数据加热的含义就是在正式部署之前，先把可能的数据先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间尽量均匀。
