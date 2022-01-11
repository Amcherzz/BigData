## 消息系统

作用：缓存

组成成分：生产者，broker，消费者

topic：主题、广播、频道

topic由分区组成

消费者组：同一个消费者组里面消费者只能消费一次消息（一次只能一个接收）

元信息：topic信息，分区信息，生产者，消费者，放在zk里面



- kafka高性能
  - 磁盘顺序写  顺序写 > 随机写
  - 日志分段存储：分区为目录，目录下存放数据  利于写入与消费竞争
  - 冗余副本  冗余分区  分区分角色：leader（承载读写压力），维护一个ISR副本列表；follower 定期向leader同步数据
  - 怎么找数据  稀疏索引 二分查找
    - index 存放position 和offset，稀疏
    - log储存数据
  - 消费者读数据  零拷贝机制 直接从内存到网卡（不经过kafka--socket--网卡）

## 源码分析

- 消费者
  - 指定broker地址 获取元数据
  - 分配消费者组
  - 反序列化  针对key和value



## 设置

### 生产者

- 参数设置

  - buffer.memory:发送缓冲区，默认32M

  - compression.type：默认为none，可以设置lz4，会加大cpu开销

  - batch.size：默认16k，根据实际数据大小，过小频繁网络请求，过大数据积压缓存区，可以配合linger.ms设置使用，比如设置100ms，发送批次的空间和时间限制

  - 可以采用异步方式发送消息

  - 分区
    - 没有设置key 轮询发送各个分区
      - 设置key，key计算hash值，相同hash值分配到同一分区
      - 特殊要求，可以自定义分区

- 一些异常

  - LeaderNotAvailableException：leader不可用

  -  NotControllerException：Controller不可用

  -  NetworkException：网络异常

  - 以上问题可以通过设置重试参数：

    ```
    props.put("retries"，10)
    props.put("retry.backoff.ms", 300)
    ```

    重试方案的问题：

    - 消息重复
    - 消息顺序打乱

    ```
    props.put("max.in.flight.requests.per.connection", 1)
    //保证生产者同一时间只能发送一条消息
    ```

- ack参数

  ```
  request.required.acks=
  ```

  - 0 只要消息发送出去就行
  - 1 保证leader中partiton写入成功，也可能出现数据丢失
  - -1保证ISR列表中所有副本写入成功

此处服务端参数设置：min.insync.replicas默认值为1，可以限制ISR列表中副本个数，当小于改值时，往该分区插入报错

- 设置kafka不丢失数据的方案
  - 副本数>=2
  - ack=-1
  - min.insync.replicas>=2

### 消费者

- 消费者组
  1. consumer要属于某个消费者组，topic的一个分区只能同时分配个消费者组中一个消费者消费，每个消费者可能有多个分区，也可能没在分配到分区
  2. 使用广播，可以设置不同消费者组
  3. 组中一个消费者挂掉，会把分区分配给组中其他消费者，达到负载均衡
- 消费者怎么读数据，根据offset，并且会持久化一个文件
  - _consumer_offsets
  - key是group.id+topic+分区号，值为当前offset值
  - 默认50个分区
- 消费者核心参数
  - fetch..max.bytes：获取一次消息大小，默认1M，可以10M
  - max.poll.records：最大返回消费条数，默认500
  - connection.max.idle.ms：设置-1时，超时不会回收网络socket连接
  - enable.auto.commit：设置为true时自动提交偏移量
  - auto.commit.interval.ms：自动提交间隔时间，默认5000ms
  - 偏移量消费策略：auto.offset.reset
    - earliest：有offset从offset消费，无则从头开始
    - latest（常用）：有offset从offset消费，无则从最新产生的数据消费
    - none：从offset从offset消费，只要有一个分区不存在已提交offset则抛异常
- 异常感知
  - 
