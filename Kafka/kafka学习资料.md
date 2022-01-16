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
- 消费者
  - 以组为单位消费数据
  - 一个消费者可以消费多个分区数据
  - 一个分区只能被一个消费者中一个消费者消费
  - kafka版本之前offset维护在zk中，之后维护在kafka内部



topic--partition  每个分区有leader和follower

生产者和消费者面向的是分区leader，只能以消费者组形式消费消息

消费者可以消费多个分区，一个分区只能被一个组中一个消费者消费



消费者的offset维护



zk确定leader和follower



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