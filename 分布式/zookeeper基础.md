# Zookeeper基础

提供分布式协调服务。

Zookeeper数据保持在内存中，用磁盘保持日志。

Zookeeper是一个目录树结构，每一个node节点最大存放1MB数据，二进制安全。

节点类型：持久节点、临时节点、序列节点。



分布式锁：临时顺序节点。

### 特点

- 顺序一致性：来自客户端的更新将按照它们发送的顺序应用。单节点读。
- 原子性
- 单一系统映射：每个节点的数据都是一致的，最终一致性。
- 可靠性：持久化
- 及时性：在特定时间范围内，各个节点的数据最终一致。



zookeeper集群Follower节点会复制Leader的内容，包括sessionId。所以每一个node中的sessionId都是一致的，复制sessionId也会消耗一次事务id。



zookeeper会开启两个端口，一个端口用来选主投票，还有一个是用来Leader接收write请求。

### HA

分布式一致性算法Paxos

ZAB原子广播协议



集群中分为主从结构（Leader和Follower），各节点间可快速复制。Zookeeper数据保持在内存中。

节点角色：Leader、Follower、Observer。

Leader负责写，Follower负责读。

当Leader挂了之后，重新选选举速度块，大概200ms。

只有Follower能参与选举。



每一个节点都会有自己的myid和维护的事务id。

新Leader选举规则（过半通过）：

1、先找数据最全的节点，即事务id最大的。

2、再看myid大小，选大的。



应用场景：

1、通过watch机制，实现应用配置信息的加载和监听。

2、分布式锁。临时节点+顺序节点，watch前一个序列的锁，释放回调压力。

