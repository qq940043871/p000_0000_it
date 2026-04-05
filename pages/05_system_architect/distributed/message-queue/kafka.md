# Kafka - 架构师学习笔记

## Kafka

Apache Kafka是一个分布式流处理平台，最初由LinkedIn开发，后来成为Apache项目。Kafka被设计为一个分布式、分区化、可复制的提交日志服务，主要用于构建实时数据管道和流应用。

## Kafka核心概念

### Topic（主题）

- Kafka中消息的分类，类似于数据库中的表。每个Topic可以分为多个Partition。

### Partition（分区）

- Topic的分区，每个Partition是一个有序的、不可变的消息序列。

### Offset（偏移量）

- 消息在Partition中的唯一标识，表示消息在Partition中的位置。

### Consumer Group（消费者组）

- 一组消费者的逻辑分组，同一个Group中的消费者不会重复消费同一条消息。

### Broker（代理）

- Kafka集群中的一个服务器节点，负责存储和传递消息。

### Zookeeper

- 分布式协调服务，负责管理Kafka集群的元数据和协调工作。

## Kafka架构

### 架构图

```
P (Producer)
  |
  v
+----------------+
|                |
|  B1 (Broker 1) |
|  B2 (Broker 2) |
|  B3 (Broker 3) |
|                |
+----------------+
  |
  v
+----------------+
|                |
|  Topic A: P0, P1 |
|  Topic B: P0, P1 |
|                |
+----------------+
  |
  v
C (Consumer)
  |
  v
ZK (Zookeeper)
```

## 消息传递模式

### 点对点模式

- 一条消息只能被一个消费者消费，消费者之间是竞争关系。

```java
// Producer
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

Producer producer = new KafkaProducer(props);
producer.send(new ProducerRecord("my-topic", "key", "value"));
producer.close();
```

### 发布/订阅模式

- 一条消息可以被多个消费者消费，消费者之间是独立的。

```java
// Consumer
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "my-group");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

Consumer consumer = new KafkaConsumer(props);
consumer.subscribe(Arrays.asList("my-topic"));

while (true) {
    ConsumerRecords records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord record : records) {
        System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
    }
}
```

## Partition与Consumer Group

### 分区分配策略

- Range策略：按照数值范围分配分区
- RoundRobin策略：轮询分配分区
- Sticky策略：尽量保持分区分配的稳定性

### 分区与消费者关系

- 一个Partition只能被同一个Consumer Group中的一个Consumer消费
- 一个Consumer可以消费多个Partition
- Consumer数量不应超过Partition数量，否则会有Consumer空闲

## 可靠性保证

### Producer可靠性

- 通过acks参数控制消息确认机制。

  - acks=0：不等待确认，性能最高但可能丢失消息
  - acks=1：等待Leader确认，性能和可靠性平衡
  - acks=all：等待所有副本确认，可靠性最高但性能最低

### Consumer可靠性

- 通过offset提交策略控制消费可靠性。

  - 自动提交：定期自动提交offset，可能重复消费
  - 手动提交：处理完消息后手动提交offset，可精确控制

## Kafka Streams

### 流处理API

- Kafka Streams是用于构建流处理应用的客户端库，允许对Kafka中的数据进行实时处理。

```java
// WordCount示例
Properties props = new Properties();
props.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-application");
props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

StreamsBuilder builder = new StreamsBuilder();
KStream source = builder.stream("input-topic");

KTable counts = source
    .flatMapValues(value -> Arrays.asList(value.toLowerCase().split(" ")))
    .groupBy((key, value) -> value)
    .count();

counts.toStream().to("output-topic", Produced.with(Serdes.String(), Serdes.Long()));

KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();
```

## 集群管理

### 集群配置

- 关键配置参数优化集群性能。

  - num.partitions：默认分区数
  - replication.factor：默认副本数
  - min.insync.replicas：最小同步副本数
  - unclean.leader.election.enable：是否允许非同步副本成为Leader

### 管理命令

- Kafka提供的命令行工具。

```bash
# 创建Topic
kafka-topics.sh --create --topic my-topic --partitions 3 --replication-factor 1 --bootstrap-server localhost:9092

# 查看Topic列表
kafka-topics.sh --list --bootstrap-server localhost:9092

# 查看Topic详情
kafka-topics.sh --describe --topic my-topic --bootstrap-server localhost:9092

# 消费消息
kafka-console-consumer.sh --topic my-topic --from-beginning --bootstrap-server localhost:9092
```

## 监控与性能调优

### 关键监控指标

| 指标 | 说明 | 监控工具 |
|------|------|----------|
| 消息吞吐量 | 每秒处理的消息数量 | Kafka Manager, Confluent Control Center |
| 延迟 | 消息从生产到消费的时间 | JMX, Prometheus |
| 磁盘使用率 | 磁盘空间使用情况 | 系统监控工具 |
| GC频率 | 垃圾回收频率和时间 | JVM监控工具 |

### 性能调优建议

- 合理设置Partition数量，平衡并发性和资源消耗
- 优化Producer和Consumer的批处理大小
- 使用压缩减少网络传输和磁盘I/O
- 合理配置JVM参数和操作系统参数
- 实施监控和告警机制

## Kafka最佳实践

- 根据业务需求合理设计Topic和Partition数量
- 设置合适的副本数以保证数据可靠性
- 实施监控和告警，及时发现和处理异常
- 定期清理过期数据，避免磁盘空间不足
- 使用消费者组实现负载均衡和容错
- 合理配置Producer和Consumer参数以优化性能
- 实施安全措施，如SSL/TLS加密和SASL认证
