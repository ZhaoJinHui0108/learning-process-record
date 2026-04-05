# Kafka 学习笔记

## 目录
1. [Kafka 简介](#kafka-简介)
2. [Kafka 使用场景](#kafka-使用场景)
3. [Kafka 核心概念](#kafka-核心概念)
4. [Python 调用 Kafka](#python-调用-kafka)
5. [Docker 部署 Kafka](#docker-部署-kafka)
6. [Kafka 日志查看](#kafka-日志查看)

---

## Kafka 简介

### 什么是 Kafka
Apache Kafka是一个分布式事件流平台，最初由LinkedIn开发，现为Apache软件基金会的开源项目。它用于构建实时数据管道和流应用程序。

### Kafka 的特点
- **高吞吐量**: 每秒可处理数百万消息
- **低延迟**: 毫秒级延迟
- **持久化**: 消息持久化到磁盘
- **分布式**: 支持集群和分区
- **可扩展**: 水平扩展
- **容错**: 支持数据复制
- **高可靠性**: 支持 Exactly-once 语义

### Kafka vs 其他消息队列
| 特性 | Kafka | RabbitMQ | ActiveMQ |
|------|-------|----------|----------|
| 吞吐量 | 极高 | 中等 | 中等 |
| 延迟 | 低 | 低 | 中等 |
| 消息持久化 | 支持 | 支持 | 支持 |
| 消息回溯 | 支持 | 不支持 | 不支持 |
| 事务支持 | 较弱 | 支持 | 支持 |
| 队列模式 | Pub/Sub | 队列+Exchange | 队列+Topic |
| 集群模式 | 原生支持 | 需要插件 | 需要插件 |

---

## Kafka 使用场景

### 1. 日志收集
```python
# 生产者端 - 发送日志
producer.send('logs', {
    'service': 'user-service',
    'level': 'ERROR',
    'message': 'Database connection failed',
    'timestamp': time.time()
})
```

### 2. 消息系统
```python
# 订单处理流水线
# 下单服务 -> 支付服务 -> 库存服务 -> 发货服务
producer.send('orders', {'order_id': '12345', 'status': 'created'})
producer.send('orders', {'order_id': '12345', 'status': 'paid'})
producer.send('orders', {'order_id': '12345', 'status': 'shipped'})
```

### 3. 用户行为追踪
```python
# 收集用户点击事件
events = [
    {'user_id': 'u1', 'action': 'click', 'page': '/home'},
    {'user_id': 'u1', 'action': 'view', 'page': '/product'},
    {'user_id': 'u1', 'action': 'add_cart', 'product': 'p123'}
]
for event in events:
    producer.send('user_events', event)
```

### 4. 系统监控
```python
# 发送系统指标
producer.send('metrics', {
    'host': 'server-01',
    'cpu': 75.5,
    'memory': 82.3,
    'disk': 65.0
})
```

### 5. 事件溯源
```python
# 存储业务事件
events = [
    {'event': 'UserCreated', 'user_id': 'u1', 'name': 'Alice'},
    {'event': 'EmailChanged', 'user_id': 'u1', 'old_email': 'a@old.com', 'new_email': 'a@new.com'},
    {'event': 'UserDeactivated', 'user_id': 'u1', 'reason': 'voluntary'}
]
for event in events:
    producer.send('domain_events', event)
```

---

## Kafka 核心概念

### Topic（主题）
```python
# 创建Topic
# 通过命令行:
# kafka-topics.sh --create --topic my-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1

# Topic配置参数
- partitions: 分区数
- replication-factor: 副本因子
- retention.ms: 消息保留时间
- cleanup.policy: 清理策略 (delete/compact)
```

### Producer（生产者）
```python
from kafka import KafkaProducer
import json

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8'),
    key_serializer=lambda k: k.encode('utf-8') if k else None
)

# 发送消息
future = producer.send('my-topic', key='user123', value={'msg': 'hello'})
# 同步等待
result = future.get(timeout=10)
```

### Consumer（消费者）
```python
from kafka import KafkaConsumer
import json

consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],
    group_id='my-consumer-group',
    value_deserializer=lambda m: json.loads(m.decode('utf-8')),
    auto_offset_reset='earliest',
    enable_auto_commit=True
)

for message in consumer:
    print(f"Message: {message.value}, Offset: {message.offset}")
```

### Consumer Group（消费者组）
```
┌─────────────────────────────────────────────────────────┐
│                    Consumer Group                        │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │ Consumer 1  │  │ Consumer 2  │  │ Consumer 3  │      │
│  │ Partition 0 │  │ Partition 1 │  │ Partition 2  │      │
│  └─────────────┘  └─────────────┘  └─────────────┘      │
└─────────────────────────────────────────────────────────┘
```

### Partition（分区）
```python
# 分区策略
# 1. 按key哈希分区
producer.send('my-topic', key='user123', value={'data': 'xxx'})
# 2. 手动指定分区
producer.send('my-topic', value={'data': 'xxx'}, partition=0)
# 3. 轮询分区
for i in range(10):
    producer.send('my-topic', value={'index': i})
```

### Offset（偏移量）
```python
# 手动提交offset
consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],
    enable_auto_commit=False
)

for message in consumer:
    process(message)
    # 手动提交
    consumer.commit()
```

### Replica（副本）
```python
# Topic副本配置
# 通过命令行:
# kafka-topics.sh --create --topic my-topic --bootstrap-server localhost:9092 \
#   --partitions 3 --replication-factor 3

# ISR (In-Sync Replicas)
# 配置:
# min.insync.replicas=2
```

### 消息语义保证
| 语义 | 说明 | 配置 |
|------|------|------|
| At-most-once | 最多一次，可能丢失消息 | `acks=0` |
| At-least-once | 至少一次，可能重复 | `acks=1` |
| Exactly-once | 精确一次 | `acks=all` + 事务 |

---

## Python 调用 Kafka

### 安装依赖
```bash
pip install kafka-python
# 或
pip install confluent-kafka
```

### 基本示例

#### 示例1: 生产者
```python
from kafka import KafkaProducer
from kafka.errors import KafkaError
import json
import time

class KafkaProducerExample:
    def __init__(self, bootstrap_servers='localhost:9092'):
        self.producer = KafkaProducer(
            bootstrap_servers=[bootstrap_servers],
            value_serializer=lambda v: json.dumps(v).encode('utf-8'),
            key_serializer=lambda k: k.encode('utf-8') if k else None,
            acks='all',
            retries=3,
            retry_backoff_ms=500
        )
    
    def send_message(self, topic, key, value):
        try:
            future = self.producer.send(topic, key=key, value=value)
            record_metadata = future.get(timeout=10)
            print(f"Message sent to {record_metadata.topic}, "
                  f"partition {record_metadata.partition}, "
                  f"offset {record_metadata.offset}")
            return True
        except KafkaError as e:
            print(f"Failed to send message: {e}")
            return False
    
    def send_batch(self, topic, messages):
        for msg in messages:
            self.producer.send(topic, value=msg)
        self.producer.flush()
        print(f"Batch of {len(messages)} messages sent")
    
    def close(self):
        self.producer.close()

if __name__ == '__main__':
    producer = KafkaProducerExample()
    
    # 发送单条消息
    producer.send_message(
        topic='test-topic',
        key='user1',
        value={'name': 'Alice', 'action': 'login', 'time': time.time()}
    )
    
    # 批量发送
    messages = [
        {'id': i, 'data': f'message {i}'} 
        for i in range(10)
    ]
    producer.send_batch('test-topic', messages)
    
    producer.close()
```

#### 示例2: 消费者
```python
from kafka import KafkaConsumer
from kafka.errors import KafkaError
import json
import time

class KafkaConsumerExample:
    def __init__(self, topic, bootstrap_servers='localhost:9092', group_id='test-group'):
        self.consumer = KafkaConsumer(
            topic,
            bootstrap_servers=[bootstrap_servers],
            group_id=group_id,
            value_deserializer=lambda m: json.loads(m.decode('utf-8')),
            auto_offset_reset='earliest',
            enable_auto_commit=True,
            auto_commit_interval_ms=1000,
            max_poll_records=100
        )
    
    def consume(self, max_messages=10, timeout=1):
        messages = []
        for i, message in enumerate(self.consumer):
            if i >= max_messages:
                break
            print(f"Topic: {message.topic}, "
                  f"Partition: {message.partition}, "
                  f"Offset: {message.offset}, "
                  f"Key: {message.key}, "
                  f"Value: {message.value}")
            messages.append(message.value)
        return messages
    
    def consume_continuous(self, callback):
        print(f"Starting to consume from topic...")
        for message in self.consumer:
            print(f"Received: {message.value}")
            callback(message.value)
    
    def close(self):
        self.consumer.close()

if __name__ == '__main__':
    consumer = KafkaConsumerExample('test-topic')
    
    # 消费消息
    messages = consumer.consume(max_messages=5)
    print(f"Consumed {len(messages)} messages")
    
    consumer.close()
```

#### 示例3: 带错误处理的完整示例
```python
from kafka import KafkaProducer, KafkaConsumer
from kafka.admin import KafkaAdminClient, NewTopic
from kafka.errors import TopicAlreadyExistsError, KafkaError
import json
import time

class KafkaManager:
    def __init__(self, bootstrap_servers='localhost:9092'):
        self.bootstrap_servers = bootstrap_servers
        self.producer = None
        self.consumer = None
    
    def create_topic(self, topic_name, num_partitions=3, replication_factor=1):
        try:
            admin = KafkaAdminClient(
                bootstrap_servers=self.bootstrap_servers
            )
            topic = NewTopic(
                name=topic_name,
                num_partitions=num_partitions,
                replication_factor=replication_factor
            )
            admin.create_topics([topic])
            print(f"Topic '{topic_name}' created successfully")
            admin.close()
            return True
        except TopicAlreadyExistsError:
            print(f"Topic '{topic_name}' already exists")
            return False
        except KafkaError as e:
            print(f"Failed to create topic: {e}")
            return False
    
    def init_producer(self):
        self.producer = KafkaProducer(
            bootstrap_servers=self.bootstrap_servers,
            value_serializer=lambda v: json.dumps(v).encode('utf-8'),
            key_serializer=lambda k: k.encode('utf-8') if k else None,
            acks='all',
            retries=3,
            retry_backoff_ms=500,
            max_in_flight_requests_per_connection=1
        )
        print("Producer initialized")
    
    def init_consumer(self, topic, group_id='default-group'):
        self.consumer = KafkaConsumer(
            topic,
            bootstrap_servers=self.bootstrap_servers,
            group_id=group_id,
            value_deserializer=lambda m: json.loads(m.decode('utf-8')),
            auto_offset_reset='earliest',
            enable_auto_commit=False
        )
        print(f"Consumer initialized for topic '{topic}'")
    
    def send_with_transaction(self, topic, messages):
        if not self.producer:
            self.init_producer()
        
        for msg in messages:
            self.producer.send(topic, value=msg)
        
        try:
            self.producer.flush()
            print(f"Sent {len(messages)} messages in transaction")
        except KafkaError as e:
            print(f"Transaction failed: {e}")
            self.producer.abort()
    
    def consume_with_retry(self, topic, group_id, max_retries=3):
        if not self.consumer:
            self.init_consumer(topic, group_id)
        
        for message in self.consumer:
            try:
                print(f"Processing message: {message.value}")
                yield message.value
                self.consumer.commit()
            except Exception as e:
                print(f"Error processing message: {e}")
                continue
    
    def close(self):
        if self.producer:
            self.producer.close()
        if self.consumer:
            self.consumer.close()
        print("Kafka manager closed")

if __name__ == '__main__':
    kafka = KafkaManager('localhost:9092')
    
    # 创建topic
    kafka.create_topic('my-topic', num_partitions=3, replication_factor=1)
    
    # 初始化生产者
    kafka.init_producer()
    
    # 发送消息
    test_messages = [
        {'id': i, 'data': f'message {i}', 'timestamp': time.time()}
        for i in range(5)
    ]
    kafka.send_with_transaction('my-topic', test_messages)
    
    # 消费消息
    for msg in kafka.consume_with_retry('my-topic', 'test-group'):
        print(f"Received: {msg}")
    
    kafka.close()
```

#### 示例4: 高级特性 - 拦截器和分区器
```python
from kafka import KafkaProducer
from kafka.processor import Processor
from kafka.partitioner import DefaultPartitioner
import json

class CustomPartitioner(DefaultPartitioner):
    def partition(self, key, all_partitions, available_partitions):
        if key:
            # 根据key的哈希值选择分区
            return int(key) % len(all_partitions)
        return 0

class CustomInterceptor:
    def on_send(self, record):
        # 在发送前添加元数据
        record.value['sent_at'] = time.time()
        record.value['client'] = 'python-producer'
        return record
    
    def on_ack(self, record, exception):
        if exception:
            print(f"Send failed: {exception}")
        else:
            print(f"Send succeeded for {record.value}")

class AdvancedProducer:
    def __init__(self):
        self.producer = KafkaProducer(
            bootstrap_servers='localhost:9092',
            value_serializer=lambda v: json.dumps(v).encode('utf-8'),
            key_serializer=lambda k: k.encode('utf-8') if k else None,
            partitioner=CustomPartitioner(),
            interceptor_classes=[CustomInterceptor]
        )
    
    def send_ordered_messages(self, topic, key_values):
        for key, value in key_values:
            future = self.producer.send(topic, key=key, value=value)
            future.add_callback(lambda r: print(f"Delivered to {r.topic}:{r.partition}:{r.offset}"))
            future.add_errback(lambda e: print(f"Error: {e}"))
        
        self.producer.flush()
    
    def close(self):
        self.producer.close()

if __name__ == '__main__':
    producer = AdvancedProducer()
    
    messages = [
        ('user1', {'action': 'login', 'data': 'a'}),
        ('user2', {'action': 'logout', 'data': 'b'}),
        ('user1', {'action': 'purchase', 'data': 'c'}),
    ]
    
    producer.send_ordered_messages('events', messages)
    producer.close()
```

---

## Docker 部署 Kafka

### 方式一: 使用 docker-compose

#### 基本配置
```yaml
# docker-compose.yml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"
    networks:
      - kafka-network

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "29092:29092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092,PLAINTEXT_INTERNAL://kafka:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: 1073741824
    networks:
      - kafka-network
    volumes:
      - kafka_data:/var/lib/kafka/data

networks:
  kafka-network:
    driver: bridge

volumes:
  kafka_data:
```

#### 高可用配置（多Broker）
```yaml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka-1:
    image: confluentinc/cp-kafka:7.4.0
    container_name: kafka-1
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092,PLAINTEXT_INTERNAL://kafka-1:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"

  kafka-2:
    image: confluentinc/cp-kafka:7.4.0
    container_name: kafka-2
    depends_on:
      - zookeeper
    ports:
      - "9093:9092"
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9093,PLAINTEXT_INTERNAL://kafka-2:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3

  kafka-3:
    image: confluentinc/cp-kafka:7.4.0
    container_name: kafka-3
    depends_on:
      - zookeeper
    ports:
      - "9094:9092"
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9094,PLAINTEXT_INTERNAL://kafka-3:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
```

#### 带 Kafka Manager 的配置
```yaml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

  kafka-manager:
    image: hlebalbez/kafka-manager:latest
    container_name: kafka-manager
    depends_on:
      - zookeeper
    ports:
      - "9000:9000"
    environment:
      ZK_HOSTS: zookeeper:2181
      APPLICATION_SECRET: random-secret
```

### 方式二: 单独容器

```bash
# 启动 Zookeeper
docker run -d \
  --name zookeeper \
  -p 2181:2181 \
  confluentinc/cp-zookeeper:7.4.0 \
  zookeeper-server-start /etc/kafka/zookeeper.properties

# 启动 Kafka
docker run -d \
  --name kafka \
  -p 9092:9092 \
  -e KAFKA_BROKER_ID=1 \
  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 \
  -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
  confluentinc/cp-kafka:7.4.0
```

### 常用 Docker 命令

```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看Kafka日志
docker logs -f kafka

# 进入Kafka容器
docker exec -it kafka bash

# 停止所有服务
docker-compose down

# 删除数据卷
docker-compose down -v

# 扩缩容
docker-compose up -d --scale kafka=3
```

### Kafka 容器内命令

```bash
# 进入容器
docker exec -it kafka bash

# 创建Topic
kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1

# 列出所有Topic
kafka-topics.sh --list --bootstrap-server localhost:9092

# 查看Topic详情
kafka-topics.sh --describe --topic test-topic --bootstrap-server localhost:9092

# 生产者测试
kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092

# 消费者测试
kafka-console-consumer.sh --topic test-topic --from-beginning --bootstrap-server localhost:9092

# 删除Topic
kafka-topics.sh --delete --topic test-topic --bootstrap-server localhost:9092

# 修改Topic
kafka-topics.sh --alter --topic test-topic --partitions 5 --bootstrap-server localhost:9092

# 查看消费者组
kafka-consumer-groups.sh --list --bootstrap-server localhost:9092

# 查看消费者组详情
kafka-consumer-groups.sh --describe --group test-group --bootstrap-server localhost:9092
```

---

## Kafka 日志查看

### Kafka 容器日志

```bash
# 查看实时日志
docker logs -f kafka

# 查看最近100行
docker logs --tail 100 kafka

# 查看指定时间范围
docker logs --since "2024-01-01T00:00:00" kafka

# 查找错误日志
docker logs kafka 2>&1 | grep -i error

# 导出日志
docker logs kafka > kafka.log 2>&1
```

### Kafka 服务日志

Kafka的日志目录结构：
```
/var/lib/kafka/
├── data/
│   ├── topic-name-0/
│   │   ├── 00000000000000000000.log
│   │   ├── 00000000000000000000.index
│   │   └── 00000000000000000000.timeindex
│   └── topic-name-1/
├── log/
│   ├── controller.log
│   ├── server.log
│   ├── state-change.log
│   └── kafka-authorizer.log
```

```bash
# 容器内查看日志目录
docker exec kafka ls -la /var/lib/kafka/log/

# 查看server.log
docker exec kafka cat /var/lib/kafka/log/server.log

# 查看controller日志
docker exec kafka cat /var/lib/kafka/log/controller.log

# 查看状态变更日志
docker exec kafka cat /var/lib/kafka/log/state-change.log
```

### JMX 和监控

```bash
# 启用JMX
docker run -d \
  --name kafka \
  -p 9092:9092 \
  -p 9999:9999 \
  -e KAFKA_JMX_PORT=9999 \
  -e KAFKA_JMX_HOSTNAME=localhost \
  confluentinc/cp-kafka:7.4.0
```

### 监控工具

```python
# 使用 kafka-python 获取监控信息
from kafka import KafkaConsumer
from kafka.admin import KafkaAdminClient

def get_kafka_consumer_lag(bootstrap_servers='localhost:9092'):
    admin = KafkaAdminClient(bootstrap_servers=bootstrap_servers)
    
    # 获取消费者组列表
    groups = admin.list_consumer_groups()
    print(f"Consumer Groups: {groups}")
    
    admin.close()

def get_topic_stats(topic_name, bootstrap_servers='localhost:9092'):
    admin = KafkaAdminClient(bootstrap_servers=bootstrap_servers)
    
    # 列出所有topic
    topics = admin.list_topics()
    print(f"All Topics: {topics}")
    
    if topic_name in topics:
        # 获取topic描述
        topic_desc = admin.describe_configs([
            {'name': topic_name}
        ])
        print(f"Topic Config: {topic_desc}")
    
    admin.close()

if __name__ == '__main__':
    get_kafka_consumer_lag()
```

### 日志级别配置

Kafka日志级别通过`log4j.properties`配置：

```properties
# log4j.properties
log4j.rootLogger=INFO, stdout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=[%d] %p %m (%c)%n

log4j.appender.kafka=org.apache.log4j.RollingFileAppender
log4j.appender.kafka.File=/var/lib/kafka/log/server.log
log4j.appender.kafka.MaxFileSize=100MB
log4j.appender.kafka.MaxBackupIndex=10

log4j.logger.kafka=INFO
log4j.logger.org.apache.kafka=INFO
```
