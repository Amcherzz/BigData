## kafka

解决hdfs并发请求问题，缓冲、解耦

## 架构

消费者拉数据

- zk
  - broker选举
- 生产者
- broker
  - 每个有唯一id
  - 多个topic 多个分区，多个副本
  - 分区可存在一个broker中，副本不能存在一个broker中
  - 生产者和消费者面向leader
  - follower同步数据，故障替代leader
  - 有个是controller，监听zk中的临时目录，维护元数据
- 消费者
  - 以组为单位消费数据
  - 一个消费者可以消费多个分区数据
  - 一个分区只能被一个消费者中一个消费者消费
  - kafka版本之前offset维护在zk中，之后维护在kafka内部



- 副本机制
  - 0.8版本之后实现，读写数据操作均在leader partition
  - follower partition从leader partition同步数据
  - leader partition会维护ISR（in sync replica）列表
  - follower超过10s没有向leader拉取数据，会从ISR中移除

- 消费者组
  - 组中负载均衡
  - 自动感知
  - 消费者，消费的位置offset，保存在一个_consumer_offsets主题中，50个分区，其中key为GTP，value为offset值，隔段时间会合并，保持最新offset数据，50个分区抗住高并发请求

消费者感知

heartbeat.interval.ms 消费者与coordinator之间心跳，例3s

session.timeout.ms kafka对消费者感知 默认10s

max.poll.interval.ms 两次拉取之间间隔，超过该值则会被剔除，例5s

topic--partition  每个分区有leader和follower

生产者和消费者面向的是分区leader，只能以消费者组形式消费消息

消费者可以消费多个分区，一个分区只能被一个组中一个消费者消费



消费者的offset维护



zk确定leader和follower

- 消费者实现rebalance

  - 每个消费者组选择一个broker为coordinator，感知组中心跳，判断宕机，启动rebalance
  - coordinator机器，groupID先hash，再与_consumer_offsets分区数取模，得到其分区号所在的broker

  1. 组中每个consumer发送JoinGroup请求至Coordinator
  2. Coordinator选择一个consumer为leader
  3. 把consumer_group情况发送给leader
  4. leader制定消费方案
  5. leader通过SyncGroup发送给Coordinator
  6. Coordinator把方案发送各个consumer，然后从指定分区开发消费数据

- rebalance分区策略

  - range 范围
  - round-robin 轮询
    - 上述方案，再分配会导致已有分区重新划分
  - sticky
    - 黏性分区，再分配保证已有分区不被打乱



### broker内部概念

LEO：log end offset最新offset + 1 ，LEO用于更新HW

HW：高水位，之前的对对消费者可见

消费者获取数据 offset 是以GTP，group-topic-partition方式维护

每一个topic的每一个分区对应一个文件，例如first有两个分区

```
drwxrwxr-x. 2 llc llc   167 Jan 16 05:50 first-0
drwxrwxr-x. 2 llc llc   167 Jan 16 05:38 first-1
```

```
$ ll
total 12
-rw-rw-r--. 1 llc llc 10485760 Jan 16 05:38 00000000000000000000.index
-rw-rw-r--. 1 llc llc      742 Jan 16 06:02 00000000000000000000.log
-rw-rw-r--. 1 llc llc 10485756 Jan 16 05:38 00000000000000000000.timeindex
-rw-rw-r--. 1 llc llc        8 Jan 16 05:50 leader-epoch-checkpoint
-rw-rw-r--. 1 llc llc       43 Jan 16 05:38 partition.metadata
```

- index 索引信息 稀疏索引，log写入4k，index记录一条索引  offset对应物理位置



数据存放在log文件中，数据以追加形式写入，每条数据都有自己的offset，消费者消费时会记录自己的offset(GTP)，以便下次再消费

为了避免log文件过大，引入分片机制，配置大小在server.properties

```
log.segment.bytes=1073741824   //默认1G
```

log达到分片大小，会创建新的log，同时命令以log对应的offset，这样可以提高查询效率，每个分片有index，而index则是稀疏索引，同时是从0开始的

- 生产者

分区原因：提高并发，方便集群扩展

- 分区原则

  - 指定p
  - 没指定p有k，k的hash与分区数取余
  - 没有指定k和p，则黏性分区
  - 说明：根据业务关键字，指定对应key，能够保证数据在对应分区内的顺序
  
  ```
   * The default partitioning strategy:
   * <ul>
   * <li>If a partition is specified in the record, use it
   * <li>If no partition is specified but a key is present choose a partition based on a hash of the key
   * <li>If no partition or key is present choose the sticky partition that changes when the batch is full.
  ```
  
  通过key取余分区
  
  ```
          // hash the keyBytes to choose a partition
          return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions
  ```
  
  生产者发送数据可靠性保证 ack (acknowledgement 确认收到)

## 性能优势

- 顺序写
- 零拷贝 直接发送给网卡 省去到页缓存  kafka-->socket-->网卡，减少上下文切换和数据复制
- 网络设计 保证了高并发，高性能原因  JAVA中NIO  **重要****



总结：

- 高并发
  - 网络架构：三层架构，多selector，多线程，队列设计（NIO）
- 高可用
  - 多副本机制
- 高性能
  - 写数据
    - 页缓存
    - 顺序写
  - 读数据
    - 稀疏索引 快速定位
    - 零拷贝  减少复制次数，减少进程与系统之家上下文切换



### 集群规模分析

每天10亿请求，二八法则

一天24小时，12点到8点2亿，剩余16小时8亿

3小时，处理6亿，6亿/3=5.5万/s  qps=5.5万

10亿*50k = 46T

2个副本，3天  46 * 2 * 3 = 276T

总结：10亿请求，高峰5.5万qps，276T数据



机器选择

1. 虚拟机和物理机，一般采用物理机
2. 高峰期qps量，5.5万一两台够，一般采用峰值的四倍，20万qps，五台物理机，每台4万qps请求
3. *硬盘 HDD和SDD，顺序写差不多，随机写SDD更好*，每台约60T，则11块7T硬盘，77 * 0.8 = 61.6T
4. 内存评估
   - 基于os cache 内存大性能好
   - 核心scala客户端java写，需要JVM，没有太多数据结构，根据经验给10G
   - 10亿请求，100主题，5分区，2副本，每个副本下文件1G，保证最新log在内存里面则共1000g，只需要最新的25%，则为250G，250g / 5 = 50g ，50 + 10 = 60g 则64g内存满足需求，128g更好
5. CPU评估，依据服务中的线程
   - acceptor线程 1
   - processor 线程 3   6-9线程
   - 处理请求线程 8   32线程
   - 定时清理线程、拉取数据线程、定时检查ISR机制等等
   - 启动kafka服务后大概100多个线程
   - 4 core  几十个打满
   - 8 core 可以几十个
   - 16 core 合适
   - 32 core 最好
6. 千兆网卡，万兆更佳
   - 5.5万qps  5.5 / 5 万 * 50k = 488M，由于副本机制 488 * 2 =  976M，千兆网卡一般无法达到峰值，700M左右，有一定压力

### kafka集群

主从架构，controller，通过zk来管理集群元数据

配置文件说明

```
vim server.properties

# 客户端监听端口号
#listeners=PLAINTEXT://:9092


# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# 消息系统即日志文件，数据存放目录，生产环境用多块硬盘，不同硬盘目录用,隔开
# A comma separated list of directories under which to store log files
log.dirs=/data0/logs,/data1/logs...

# 默认为1 一般再创建topic时，指定
# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# os cache刷入磁盘条件
# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

# 默认生命周期，默认7天，有3天或12小时的
# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168
```

集群搭建完成后需压力测试，kafka运维管理可用kafka-manager



### 提高吞吐率

参数设置

- buffer.memory 缓冲区大小 默认32M
- compression.type压缩类型 默认none，可选lz4，提高吞吐，加大生产者CPU开销
- batch.size小频繁网络请求，大缓冲区压力，默认16k，生产环境可据实际情况提高，配合linger.ms使用，默认是0，可以设置100ms，满足容量和时间条件之一即可发送



### 一些问题说明

- 消息重复
  - leader切换，重试导致消息重复
- 消息乱序
  - 前面的消息发送失败重试导致，设置max.in.flight.requests.per.connection参数设置为1，保证同一时间只能发一条消息
  - 可以设置重试时间间隔，默认100ms，retry.backoff.ms
  - 开发中，重试可解决大多数问题



### producer端设置

request.required.ack

- 0 只管发送，性能好
- 1 需leader写入成功，可能丢数据
- -1 需ISR中所有副本写入成功

kafka服务端

min.insync.replicas，默认1，参数限制ISR列表中副本数，当实际副本数，小于设置值再往对应分区写数据会报错



不丢数据方案：

1. 分区副本 >= 2
2. ack = -1
3.  min.insync.replicas >= 2

## kafka安装



```
# 更改kafka消息即log存放目录
vi server.properties

# 注意更改每个节点的id
broker.id=0
# 更改消息存放目录
log.dirs=/opt/module/kafka/logs

# 启动集群
1. 先启动zk

2. 启动kafka
kafka-server-start.sh -daemon config/server.properties

3. 关闭kafka
bin/kafka-server-stop.sh stop
```

- 集群启动脚本

```shell
#!/bin/bash
if [ $# -lt 1 ]
then 
  echo "Input Args Error....."
  exit
fi
for i in hadoop1 hadoop2 hadoop3
do

case $1 in
start)
  echo "==================START $i KAFKA==================="
  ssh $i /opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties
;;
stop)
  echo "==================STOP $i KAFKA==================="
  ssh $i /opt/module/kafka/bin/kafka-server-stop.sh stop
;;

*)
 echo "Input Args Error....."
 exit
;;  
esac
done
```

**特别注意：zk需在kafka关闭后再关闭**

## 常用命令

- 命令使用帮助

  命令行主要用于学习和测试，直接输入kafka的bin目录中sh脚本敲回车，会显示如何使用

- 查看分区列表

  ```
  kafka-topics.sh --list --bootstrap-server hadoop1:9092
  ```

- 创建topic

  ```
  kafka-topics.sh --create --bootstrap-server hadoop1:9092 --topic first_topic_name --partitions 2 --replication-factor 3
  ```

- 查看信息

  ```
  kafka-topics.sh --describe --bootstrap-server hadoop1:9092 --topic first_topic_name
  Topic: first_topic_name	TopicId: GwzC6MEiTJ2PfpvrZk7Etg	PartitionCount: 2	ReplicationFactor: 3	Configs: segment.bytes=1073741824
  	Topic: first_topic_name	Partition: 0	Leader: 0	Replicas: 0,2,1	Isr: 0,2,1
  	Topic: first_topic_name	Partition: 1	Leader: 2	Replicas: 2,1,0	Isr: 2,1,0
  ```

- 修改分区数(只能改大，不能改小)

  ```
  kafka-topics.sh --alter --bootstrap-server hadoop1:9092 --topic first_topic_name --partitions 3
  ```

- 删除主题

  ```
  kafka-topics.sh --delete --bootstrap-server hadoop1:9092 --topic first_topic_name
  ```

- 生产者

  ```
  kafka-console-producer.sh --bootstrap-server hadoop1:9092 --topic first
  ```

- 消费者

  ```
  kafka-console-consumer.sh --bootstrap-server hadoop1:9092 --topic first
  ```

  注意再次在新打开上述，依然可以消费，因为属于新的消费者组，同时重置offset，不能获得之前的数据，除非加入--from-beginning，这样获取的数据只能保证分区有序，并不能保证整体有序，生产同时进行消费，能够保证有序



```
# 生产者
$ kafka-console-producer.sh --bootstrap-server hadoop1:9092 --topic first
>4444
>666
>hello
>kafka
>nice 
>to
>see
>you
>

# 消费者
$ kafka-console-consumer.sh --bootstrap-server hadoop1:9092 --topic first --from-beginning
4444
hello
nice 
see
666
kafka
to
you
```

指定消费者组

```
# 参数传入
kafka-console-consumer.sh --bootstrap-server hadoop1:9092 --topic first --group [groupid]

# 用配置文件
kafka-console-consumer.sh --bootstrap-server hadoop1:9092 --topic first --consumer.config /opt/module/kafka/config/consumer.properties
```

由于指定了消费者组，同时有两个分区，如果上述执行两次，那么相当于一个消费者组里面两个消费者，刚好每个消费者消费一个分区