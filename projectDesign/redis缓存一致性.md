### 一、背景

当redis作为缓存使用时，存在redis和db的数据不一致问题。

#### 流程

读的时候，先读redis，如果redis中存在，直接返回；如果redis中没有就从db中读取数据，同时更新redis。

只读数据不存在数据不一致的情况。

更新数据的时候，存在数据不一致的情况。

1、先更新redis，再更新db

问题：如果更新db失败，redis与db数据不一致。

2、先删除redis，再更新db。

问题：如果更新db时有请求过来，此时会导致redis更新成了旧数据。

### 二、解决方案

#### 延时双删

先删除redis缓存，再更新db数据，在更新db后再删除redis。

#### RocketMQ半消息机制（推荐）

1. A向RocketMQ发送一条half Message给B。

2. A更新Redis中的数据。如果更新成功，A向MQ发送commit；否则发送rollback。
3. 如果A超时等MQ得不到回应，half message会回查A操作的状态。
4. MQ判断是否发送message给B，B负责更新DB数据。