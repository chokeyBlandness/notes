

**磁盘**：
寻址：ms级。带宽：G/M

**内存**：
寻址：ns级。带宽：很大。
磁盘比内存在寻址上慢了10W倍。


## 一、数据类型

Strings、Lists、Hashes、Sets、Sorted sets、BitMaps和HyperLogLogs

`type key`指令可查看key下value的类型

### 1、String

二进制安全，存字节流。

当value为数值时，其type仍然为string，但是通过`object encoding key`指令查看其编码，显示为`int`，方便incr、decr等数值操作。

字符串的encoding为embstr。

### 2、Bitmap

`setbit key offset value`。offset为bit位从左到右的偏移位。

`bitcount key [start end]`。统计bitmap中1的数量。

`bitop operation destkey key1 [key2...]`。位操作。例如`bitop and andkey k1 k2`，将k1和k2按位与操作，赋值给andkey。

#### 使用场景

1、有用户系统，统计用户登录天数，且窗口随机

用365个bit（46bytes）就能表示一个用户一年的登录情况。

2、假设系统有上亿用户，统计系统的活跃用户

用时间做key，每一个用户映射到一个唯一的二进制位，进行统计。

```sh
setbit 20220601 1 1
setbit 20220602 1 1
setbit 20220602 7 1
bitop or destkey 20220601 20220602
bitcount destkey 0 -1
```

### 3、List

可重复有序的。

`lpush key value [v2...]`。从左边依次添加list。

`rpush key value [v2...]`。从右边依次添加list。

`lpop key`。从左边弹出数据。

list可以实现栈和队列。

`lrange key [start end]`。查看list中数据。

`blpop key timeout`。阻塞弹出。

### 4、Hash

`set key::field value`或`hset key field value`。新增hashmap。

### 5、Set

去重、无序的

`sadd key value [value2..]`。新增set数据。

`sinter key1 key2`或`sinterstore destkey key1 key2`。求交集

`sunion key1 key2`。求并集

`sdiff key1 key2`。求差集

`srandmember key count`。随机返回集合中的元素。如果count为正数且小于集合基数，随机返回count个元素的集合；如何count大于集合基数，返回整个集合；如何count为负数，返回元素可能会重复出现多次，数组长度为count的绝对值。

srandmember可用于抽奖：例如10个奖，抽奖用户大于10或小于10，中奖重复或不重复。

### 6、SortedSet

有序无重复的集合

`zadd key score member`。添加有序集合元素。
物理内存存放左小右大，不随zrang、zrevrang等命令发生变化。

排序是怎么实现的？如何提高增删改的速度？skiplist跳跃表

## 二、消息订阅、事务等

### 1、管道pipeline

将一系列指令，通过管道的方式，一次性发送给redis。降低了通讯的成本。

### 2、发布订阅

`publish channel message`。向channel发布消息。

`subscribe channel message`。订阅监听消息，接收到的消息是订阅后发布的。

### 3、事务

MULTI、EXEC、DISCARD、WATCH是事务相关的指令。

redis不支持回滚。

### 4、布隆过滤器

需安装扩展支持。用来解决缓存穿透的问题。

### 5、缓存与数据库

redis既可以做缓存，也可以做数据库。

缓存与数据库的区别：缓存数据不重要，不是全量数据，应随访问变化。内存大小是有限的。

#### 回收策略

配置项maxmemory指定内存大小。

maxmemory-policy指定内存回收策略。noeviction、allkeys-lru、volatile-lru、allkeys-random、volatile-random、volatile-ttl。

#### 过期策略

读取数据时并不会延长过期时间，发生写时，会改变过期时间。

过期策略：

1. 被动（惰性）。在被访问时，检查是否过期，并删除过期key。
2. 主动（定期）。每隔一定时间随机抽取一些设置了过期时间的key，检查删除过期key。

稍微牺牲了内存，但提高了性能。

## 三、持久化

#### RDB快照/副本

有时点性。

bgsave，fork()出一个子进程进行持久化，此时效率比较高，同时也能对外提供服务；

save，阻塞式，在关机维护时使用。

弊端：只有一个dump.rdb。数据容易丢失，例如8点得到一个rdb，9点redis挂了，8-9的数据丢失。

#### AOF

redis的写操作记录到文件中。

数据丢失少。

弊端：体量会无限变大，恢复数据比较慢。

**对aof无限变大问题的优化**

1. 4.0之前，重写aof，删除抵消的命令，合并重复的命令，最终得到一个体量小的纯指令日志文件。
2. 4.0之后，重写aof，将老的数据RDB到AOF文件中，再将增量指令append到AOF，最终得到一个混合体。

AOF和RDB可同时开启混合使用。

配置开启aof：appendonly yes

aof持久化级别：always、everysec、no

## 四、集群

单机单节点问题：1、单点故障。2、容量有限。3、压力大。

AFK拆分原则

X轴：全量镜像，主备，读写分离。

Y轴：按功能业务划分不同的多个redis实例。

Z轴：按优先级、逻辑再拆分。

### 主从同步方式

redis默认使用异步复制，其特点是低延迟和高性能。

1、同步方式、阻塞方式

强一致性，破坏可用性。向主机发送数据时，所有备机同步之后返回客户端操作成功。

2、异步方式

弱一致性，容忍一定的数据丢失。主机向备机异步同步，有一台成功，就返回客户端操作成功。

3、消息队列

最终一致性。主机通过将数据发到消息队列的方式，进行备机的同步。

### 高可用

对主机做高可用。

sentinel哨兵。监控、通知、自动故障转移。
