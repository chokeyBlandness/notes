# RocketMQ基础

## 一、安装

windows上安装，需要先配置`JAVA_HOME`和`ROCKETMQ_HOME`。

JAVA_HOME在安装jdk时就应该已经配好。ROCKET_HOME配置为rocketmq的项目目录。

### 1、启动nameServer

先启动nameserver，`bin/mqnamesrv`。本质调用了`bin/runserver.sh`，该脚本中可配置jvm启动参数。默认端口9876

在conf目录下新增一个namesrv.properties文件，可配置端口等信息。

```properties
listenPort=9876
```

启动脚本

```shell
nohup ./bin/mqnamesrv -c ./conf/namesrv.properties >namesrv.out &
```

### 2、启动broker

再启动broker，`bin/mqbroker.cmd`。本质调用了`bin/runbroker.sh`，该脚本中可配置jvm启动参数。默认端口10911。

可通过指令指定nameserver，`./bin/mqbroker -n localhost:9876`。

在conf目录下有broker的默认配置文件broker.conf，也可以新增一个broker.properties文件进行指定，可配置端口、nameserver等信息。

```properties
listenPort=10911
namesrvAddr=localhost:9876
brokerClusterName=cluster01
brokerName=broker-a
# master是0，slave是1
brokerId=0
# 刷盘模式，异步刷盘，降低机器的磁盘io瓶颈
flushDiskType=ASYNC_FLUSH
# 数据默认在~/store目录下
storePathRootDir=/var/rocketmq/store
storePathCommitLog=/var/rocketmq/store/commitlog
storePathIndex=/var/rocketmq/store/index
storePathConsumeQueue=/var/rocketmq/store/consumequeue
```

修改log日志地址（默认在~/logs目录下）

```shell
sed -i 's#${user.home}#/var/rocketmq#g' *.xml
```

启动脚本

```shell
nohup ./bin/mqbroker -c ./conf/broker.properties >broker.out &
```

### 3、命令行管理工具

mqadmin.cmd

创建topic指令

```shell
./mqadmin updatetopic -c cluster01 -t blandTopic01 -n localhost:9876
```

可以配置queue的读写权限。

**读写队列数量不一致的使用场景**

做消息队列数的压缩，例如从3个队列减少为2个，先让一个队列只读不写，继续消费，消费完了之后关闭，避免消息丢失。

## 二、消息的使用

底层采用了netty进行通信。

采取工作线程池、IO线程池进行处理，使用线程池进行资源隔离。

为什么要把所有的消息都放在一个commitLog中？

磁盘读写中，顺序读写的性能是最高的。

事务消息发送原理：

发送事务消息到broker，broker将message先进行包装，拿出topic、queueId等信息封装到property中，将这个封装后的消息存到专门的一个事务半消息队列中。具体代码`TransactionalMessageBridge.parseHalfMessageInner(MessageExtBrokerInner)`。所以事务的halfMessage也是会存在commitLog中。

延迟消息发送原理：

和事务消息类似，将message发送到对应的ScheduleQueue，每个级别的delayLevel都会创建一个队列。

消息的push和pull，本质都是pull。push就是轮询去pull。



重试队列

消息被重试消费时，会将消息放在一个RETRY+consumerGropu为topic的重试队列中。重试队列也是延迟队列，先把重试消息放在延迟队列，再放到真正的retry重试队列。

死信队列

消息被重试消费多次仍无法成功时，会将消息放在一个DLQ+consumerGropu为topic的死信队列中。该队列的消息需要手动去消费。



