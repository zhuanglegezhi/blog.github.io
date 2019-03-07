# 生产者

参数解析

| Property   | Describe                                                     | Default |
| ---------- | ------------------------------------------------------------ | ------- |
| acks       | producer需要server接收到数据之后发出的确认接收的信号，此项配置就是指procuder需要多少个这样的确认信号。<br/>0： producer 不会等待副本的任何响应，这样最容易丢失消息但同时性能却是最好的！
1：这是一种折中的方案，它会等待副本 Leader 响应，但不会等到 follower 的响应。一旦 Leader 挂掉消息就会丢失。但性能和消息安全性都得到了一定的保证。
-1/all：会确保所有的 follower 副本都完成数据的写入才会返回。但同时性能和吞吐量却是最低的。 | 1       |
| batch.size | producer将试图批处理消息记录，以减少请求次数。这将改善client与server之间的性能。这项配置控制默认的批量处理消息字节数。<br/>不会试图处理大于这个字节数的消息字节数。
发送到brokers的请求将包含多个批量处理，其中会包含对每个partition的一个请求。
较小的批量处理数值比较少用，并且可能降低吞吐量（0则会仅用批量处理）。较大的批量处理数值将会浪费更多内存空间，这样就需要分配特定批量处理数值的内存大小。 | 16384   |
| retries    | 设置大于0的值将使客户端重新发送任何数据，一旦这些数据发送失败。允许重试将潜在的改变数据的顺序，如果这两个消息记录都是发送到同一个partition，则第一个消息失败第二个发送成功，则第二条消息会比第一条消息出现要早。 | 0       |





![kafka-producer](/Users/zz/coding/git_project/zhuanglegezhi/blog.github.io/resource/kafka-producer.png)



1. ProducerInterceptors对消息进行拦截。
2. Serializer对消息的key和value进行序列化。
3. Partitioner为消息选择合适的Partition。
4. RecordAccumulator收集消息，实现批量发送。
5. Sender从RecordAccumulator获取消息。
6. 构造ClientRequest。
7. 将ClientRequest交给NetworkClient,准备发送。
8. NetworkClient将请求送入KafkaChannel的缓存。
9. 执行网络I/O,发送请求。
10. 收到响应，调用ClientRequest的回调函数。
11. 调用RecordBatch的回调函数，最终调用每个消息上注册的回调函数