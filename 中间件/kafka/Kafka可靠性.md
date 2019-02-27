# Kafka可靠性

Kafka体系架构

![kafka体系架构](/Users/zz/coding/git_project/zhuanglegezhi/blog.github.io/resource/kafka体系架构.png)

副本（replication）

在Kafka中发生复制时确保partition的日志能有序地写到其他节点上，N个replicas中，其中一个replica为leader，其他都为follower, leader处理partition的所有读写请求，与此同时，follower会被动定期地去复制leader上的数据。![kafka副本](/Users/zz/coding/git_project/zhuanglegezhi/blog.github.io/resource/kafka副本.png)

ISR（In-Sync Replicas，副本同步队列）