# MQ (消息队列) 学习笔记

## 目录
1. [MQ 简介](#mq-简介)
2. [MQ 使用场景](#mq-使用场景)
3. [MQ 核心概念](#mq-核心概念)
4. [RabbitMQ 详解](#rabbitmq-详解)
5. [Python 调用 MQ](#python-调用-mq)
6. [Docker 部署 RabbitMQ](#docker-部署-rabbitmq)
7. [MQ 日志查看](#mq-日志查看)

---

## MQ 简介

### 什么是消息队列
消息队列（Message Queue，简称MQ）是一种进程间通信或同一进程的不同线程间的通信方式。消息队列就是消息的链表队列，具有FIFO（先进先出）的特性。

### 常见 MQ 产品
| 名称 | 开发语言 | 特点 | 适用场景 |
|------|----------|------|----------|
| **RabbitMQ** | Erlang | 功能丰富，支持多种协议 | 中小型系统 |
| **Apache RocketMQ** | Java | 高吞吐，高可靠 | 电商、金融 |
| **Apache ActiveMQ** | Java | 老牌，稳定 | 中小型系统 |
| **ZeroMQ** | C++ | 轻量级，高性能 | 低延迟场景 |
| **NATS** | Go | 极简，高性能 | 云原生 |

### MQ 的优势
- **异步处理**: 提高系统响应速度
- **解耦**: 降低系统间耦合度
- **削峰填谷**: 缓解高并发压力
- **可靠传输**: 保证消息可靠传递
- **顺序保证**: 保证消息处理顺序

### MQ 的劣势
- **系统复杂度增加**: 需要考虑消息丢失、重复等问题
- **系统可用性降低**: MQ宕机会影响系统
- **一致性难保证**: 分布式事务问题

---

## MQ 使用场景

### 1. 异步处理
```
传统方式:                            使用MQ:
用户注册 ──> 发送邮件 ──> 发送短信     用户注册 ──> 写库
           ──> 发送通知                 │
           (等待所有操作完成)            ▼
                                     发送邮件
                                     发送短信
                                     发送通知
                                   (并行处理)
```

### 2. 应用解耦
```
传统方式:                            使用MQ:
订单系统 ──> 库存系统               订单系统 ──> MQ ──> 库存系统
      ──> 物流系统                         ──> 物流系统
      ──> 积分系统                         ──> 积分系统
      ──> 通知系统                         ──> 通知系统
(强耦合)                           (松耦合)
```

### 3. 流量削峰
```
                ┌─────────┐
请求 ──────────▶│   MQ    │─────────▶ 处理系统
(10000/s)       │(缓冲)    │          (100/s)
                └─────────┘
              峰值请求被缓存
              慢慢消费处理
```

### 4. 日志处理
```python
# 应用将日志发送到MQ
producer.send('logs', {'level': 'INFO', 'message': 'User logged in'})
```

### 5. 消息通知
```python
# 订单状态变更通知
producer.send('notifications', {
    'user_id': '12345',
    'type': 'email',
    'subject': 'Order Confirmed',
    'content': 'Your order #12345 has been confirmed'
})
```

---

## MQ 核心概念

### 消息模式

#### 点对点模式 (Point-to-Point)
```
┌─────────┐     ┌─────────────┐     ┌─────────┐
│ Producer │────▶│    Queue    │────▶│ Consumer│
└─────────┘     └─────────────┘     └─────────┘
                      │
                      ▼
                 消息被消费后
                  从队列删除
```

#### 发布/订阅模式 (Pub/Sub)
```
┌─────────┐     ┌─────────────┐     ┌──────────────┐
│ Producer │────▶│   Exchange  │────▶│  Consumer 1  │
└─────────┘     └─────────────┘     └──────────────┘
                    │
                    │ fanout
                    ▼
              ┌─────────────┐
              │  Consumer 2 │
              └─────────────┘
```

### 消息组成
```
┌─────────────────────────────────────┐
│            Message                  │
├─────────────┬───────────────────────┤
│ Header      │ Properties            │
│ - routing   │ - content_type        │
│ - priority  │ - delivery_mode       │
│ - message_id│ - timestamp           │
├─────────────┴───────────────────────┤
│             Body                    │
│         (实际的消息内容)              │
└─────────────────────────────────────┘
```

### 消息确认机制
| 模式 | 说明 | 可靠性 |
|------|------|--------|
| Auto Ack | 消息投递即确认 | 可能丢失 |
| Manual Ack | 消费者手动确认 | 可靠 |
| Negative Ack | 拒绝消息，可重试 | 可靠 |

### 消息持久化
```python
# RabbitMQ
channel.basic_publish(
    exchange='',
    routing_key='queue_name',
    body=message,
    properties=pika.BasicProperties(
        delivery_mode=2  # 持久化
    )
)
```

### 死信队列 (DLQ)
```python
# 消息被拒绝且不重新入队
# 消息超时未消费
# 队列达到最大长度
# 消息进入死信队列
```

---

## RabbitMQ 详解

### RabbitMQ 架构
```
┌─────────────────────────────────────────────────────────┐
│                      RabbitMQ Server                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐         │
│   │ Exchange │───▶│  Queue   │───▶│ Consumer │         │
│   │ (类型)   │    │ (存储)   │    │ (消费者)  │         │
│   └──────────┘    └──────────┘    └──────────┘         │
│        │                                    ▲           │
│        │                                    │           │
│        ▼                                    │           │
│   ┌──────────┐                             │           │
│   │ Producer │                             │           │
│   │ (生产者)  │                             │           │
│   └──────────┘                             │           │
│                                              │           │
└──────────────────────────────────────────────┘           │
```

### Exchange 类型

#### Direct Exchange
```python
# 精确匹配 routing_key
channel.exchange_declare(exchange='direct_exchange', exchange_type='direct')
channel.queue_bind(queue='queue_name', exchange='direct_exchange', routing_key='key1')
```

#### Fanout Exchange
```python
# 广播到所有绑定的队列
channel.exchange_declare(exchange='fanout_exchange', exchange_type='fanout')
# 所有绑定的队列都会收到消息
```

#### Topic Exchange
```python
# 通配符匹配
# * 匹配一个单词
# # 匹配零个或多个单词
channel.queue_bind(queue='queue_name', exchange='topic_exchange', routing_key='user.*')
channel.queue_bind(queue='queue_name', exchange='topic_exchange', routing_key='user.#')
```

#### Headers Exchange
```python
# 根据消息头匹配
channel.queue_bind(
    queue='queue_name',
    exchange='headers_exchange',
    arguments={'x-match': 'all', 'format': 'json'}
)
```

### Virtual Host (虚拟主机)
```python
import pika

credentials = pika.PlainCredentials('guest', 'guest')
parameters = pika.ConnectionParameters(
    host='localhost',
    port=5672,
    virtual_host='/my_vhost'  # 虚拟主机
)
connection = pika.BlockingConnection(parameters)
```

---

## Python 调用 MQ

### 安装
```bash
pip install pika
pip install aio-pika  # 异步版本
```

### RabbitMQ 基本操作

#### 示例1: 简单生产者和消费者
```python
import pika
import json
import time

class RabbitMQProducer:
    def __init__(self, host='localhost', port=5672, username='guest', password='guest'):
        self.credentials = pika.PlainCredentials(username, password)
        self.parameters = pika.ConnectionParameters(
            host=host,
            port=port,
            credentials=self.credentials
        )
        self.connection = None
        self.channel = None
    
    def connect(self):
        self.connection = pika.BlockingConnection(self.parameters)
        self.channel = self.connection.channel()
        print("Connected to RabbitMQ")
    
    def declare_queue(self, queue_name, durable=True):
        self.channel.queue_declare(queue=queue_name, durable=durable)
    
    def publish(self, queue_name, message, routing_key=None):
        if isinstance(message, dict):
            message = json.dumps(message)
        
        self.channel.basic_publish(
            exchange='',
            routing_key=routing_key or queue_name,
            body=message,
            properties=pika.BasicProperties(
                delivery_mode=2,  # 持久化
                content_type='application/json'
            )
        )
        print(f"Published: {message}")
    
    def close(self):
        if self.connection:
            self.connection.close()
            print("Connection closed")


class RabbitMQConsumer:
    def __init__(self, host='localhost', port=5672, username='guest', password='guest'):
        self.credentials = pika.PlainCredentials(username, password)
        self.parameters = pika.ConnectionParameters(
            host=host,
            port=port,
            credentials=self.credentials
        )
        self.connection = None
        self.channel = None
        self.queue_name = None
    
    def connect(self):
        self.connection = pika.BlockingConnection(self.parameters)
        self.channel = self.connection.channel()
        print("Connected to RabbitMQ")
    
    def declare_queue(self, queue_name, durable=True):
        self.channel.queue_declare(queue=queue_name, durable=durable)
        self.queue_name = queue_name
    
    def callback(self, ch, method, properties, body):
        print(f"Received: {body.decode()}")
        print(f"Delivery Tag: {method.delivery_tag}")
        ch.basic_ack(delivery_tag=method.delivery_tag)
    
    def consume(self, queue_name, prefetch_count=1):
        self.declare_queue(queue_name)
        self.channel.basic_qos(prefetch_count=prefetch_count)
        self.channel.basic_consume(
            queue=queue_name,
            on_message_callback=self.callback,
            auto_ack=False
        )
        print(f"Waiting for messages on queue: {queue_name}")
        self.channel.start_consuming()
    
    def close(self):
        if self.connection:
            self.connection.close()
            print("Connection closed")


if __name__ == '__main__':
    queue_name = 'test_queue'
    
    # 生产者
    producer = RabbitMQProducer()
    producer.connect()
    producer.declare_queue(queue_name)
    
    for i in range(5):
        producer.publish(queue_name, {'id': i, 'message': f'Message {i}'})
        time.sleep(0.5)
    
    producer.close()
    
    print("\n--- Consumer would receive messages ---\n")
```

#### 示例2: 交换机和路由
```python
import pika
import json

class RabbitMQExchange:
    def __init__(self, host='localhost', port=5672):
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters(host=host, port=port)
        )
        self.channel = self.connection.channel()
    
    def setup_exchange(self, exchange_name, exchange_type='direct'):
        self.channel.exchange_declare(
            exchange=exchange_name,
            exchange_type=exchange_type,
            durable=True
        )
        print(f"Exchange '{exchange_name}' created")
    
    def setup_queues_and_bindings(self):
        # 创建多个队列
        self.channel.queue_declare(queue='queue_info', durable=True)
        self.channel.queue_declare(queue='queue_warning', durable=True)
        self.channel.queue_declare(queue='queue_error', durable=True)
        
        # 绑定队列到交换机
        self.channel.queue_bind(exchange='logs', queue='queue_info', routing_key='info')
        self.channel.queue_bind(exchange='logs', queue='queue_warning', routing_key='warning')
        self.channel.queue_bind(exchange='logs', queue='queue_error', routing_key='error')
        
        print("Queues and bindings created")
    
    def publish_log(self, level, message):
        log_message = json.dumps({'level': level, 'message': message, 'time': time.time()})
        self.channel.basic_publish(
            exchange='logs',
            routing_key=level,
            body=log_message,
            properties=pika.BasicProperties(delivery_mode=2)
        )
        print(f"Published {level} log: {message}")
    
    def close(self):
        self.connection.close()


if __name__ == '__main__':
    import time
    
    rabbitmq = RabbitMQExchange()
    
    # 设置交换机
    rabbitmq.setup_exchange('logs', 'direct')
    
    # 设置队列和绑定
    rabbitmq.setup_queues_and_bindings()
    
    # 发布日志消息
    rabbitmq.publish_log('info', 'Application started')
    rabbitmq.publish_log('warning', 'Memory usage high')
    rabbitmq.publish_log('error', 'Connection failed')
    
    rabbitmq.close()
```

#### 示例3: 发布订阅 (Fanout)
```python
import pika
import json
import threading

class PubSubExample:
    def __init__(self, host='localhost', port=5672):
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters(host=host, port=port)
        )
        self.channel = self.connection.channel()
    
    def setup_fanout_exchange(self, exchange_name):
        self.channel.exchange_declare(
            exchange=exchange_name,
            exchange_type='fanout',
            durable=True
        )
        
        # 创建两个队列用于测试
        result1 = self.channel.queue_declare(queue='subscriber1', exclusive=False)
        result2 = self.channel.queue_declare(queue='subscriber2', exclusive=False)
        
        # 绑定到交换机
        self.channel.queue_bind(exchange=exchange_name, queue='subscriber1')
        self.channel.queue_bind(exchange=exchange_name, queue='subscriber2')
        
        print(f"Fanout exchange '{exchange_name}' setup with 2 subscribers")
    
    def publish(self, exchange_name, message):
        self.channel.basic_publish(
            exchange=exchange_name,
            routing_key='',
            body=json.dumps(message)
        )
        print(f"Published to fanout exchange: {message}")
    
    def subscribe(self, queue_name, callback):
        self.channel.basic_consume(
            queue=queue_name,
            on_message_callback=callback,
            auto_ack=True
        )
        self.channel.start_consuming()
    
    def close(self):
        self.connection.close()


if __name__ == '__main__':
    pubsub = PubSubExample()
    
    # 设置fanout交换机
    pubsub.setup_fanout_exchange('notifications')
    
    # 发布消息，所有订阅者都会收到
    pubsub.publish('notifications', {'type': 'alert', 'message': 'System update available'})
    pubsub.publish('notifications', {'type': 'news', 'content': 'New features released'})
    
    pubsub.close()
    print("Pub/Sub example completed")
```

#### 示例4: 消息持久化和确认
```python
import pika
import json
import time

class ReliableProducer:
    def __init__(self, host='localhost', port=5672):
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters(host=host, port=port, heartbeat=600)
        )
        self.channel = self.connection.channel()
    
    def publish_with_confirmation(self, queue_name, message, exchange=''):
        # 开启publisher confirms
        self.channel.confirm_delivery()
        
        self.channel.queue_declare(queue=queue_name, durable=True)
        
        try:
            self.channel.basic_publish(
                exchange=exchange,
                routing_key=queue_name,
                body=json.dumps(message),
                properties=pika.BasicProperties(
                    delivery_mode=2,  # 消息持久化
                    content_type='application/json',
                    message_id=str(time.time()),
                    timestamp=int(time.time())
                ),
                mandatory=True  # 消息必须路由到队列
            )
            print(f"Message confirmed: {message}")
            return True
        except Exception as e:
            print(f"Message failed: {e}")
            return False
    
    def close(self):
        self.connection.close()


class ReliableConsumer:
    def __init__(self, host='localhost', port=5672):
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters(host=host, port=port)
        )
        self.channel = self.connection.channel()
        self.processed = []
    
    def consume_with_manual_ack(self, queue_name):
        self.channel.queue_declare(queue=queue_name, durable=True)
        
        # 设置prefetch_count，每次只获取一条消息
        self.channel.basic_qos(prefetch_count=1)
        
        def callback(ch, method, properties, body):
            message = json.loads(body)
            print(f"Received: {message}")
            
            try:
                # 处理消息
                self.process_message(message)
                # 确认消息
                ch.basic_ack(delivery_tag=method.delivery_tag)
                print(f"ACKed: {message}")
            except Exception as e:
                print(f"Error processing: {e}")
                # 拒绝消息，重新入队
                ch.basic_nack(delivery_tag=method.delivery_tag, requeue=True)
        
        self.channel.basic_consume(
            queue=queue_name,
            on_message_callback=callback,
            auto_ack=False
        )
        
        print(f"Waiting for messages on {queue_name}...")
        self.channel.start_consuming()
    
    def process_message(self, message):
        # 模拟处理
        print(f"Processing: {message}")
        time.sleep(1)
        self.processed.append(message)
    
    def close(self):
        self.connection.close()
        print(f"Processed {len(self.processed)} messages")


if __name__ == '__main__':
    queue_name = 'reliable_queue'
    
    # 生产者发送消息
    producer = ReliableProducer()
    for i in range(3):
        producer.publish_with_confirmation(queue_name, {'id': i, 'data': f'msg_{i}'})
    producer.close()
    
    print("\n--- Consumer would process messages ---\n")
```

#### 示例5: 死信队列
```python
import pika
import json
import time

class DLQExample:
    def __init__(self, host='localhost', port=5672):
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters(host=host, port=port)
        )
        self.channel = self.connection.channel()
    
    def setup_with_dlq(self):
        # 声明死信队列
        self.channel.queue_declare(queue='dlq', durable=True)
        
        # 声明主队列，配置死信交换机
        args = {
            'x-dead-letter-exchange': 'dlx',
            'x-dead-letter-routing-key': 'dlq',
            'x-message-ttl': 5000  # 5秒后过期
        }
        self.channel.queue_declare(queue='main_queue', durable=True, arguments=args)
        
        # 声明死信交换机并绑定
        self.channel.exchange_declare(exchange='dlx', exchange_type='direct')
        self.channel.queue_bind(exchange='dlx', queue='dlq', routing_key='dlq')
        
        print("Main queue with DLQ setup completed")
    
    def publish_to_main(self, message):
        self.channel.basic_publish(
            exchange='',
            routing_key='main_queue',
            body=json.dumps(message),
            properties=pika.BasicProperties(delivery_mode=2)
        )
        print(f"Published to main queue: {message}")
    
    def consume_from_dlq(self):
        def callback(ch, method, properties, body):
            print(f"DLQ Received: {body.decode()}")
            print(f"Original message ID: {properties.message_id}")
            ch.basic_ack(delivery_tag=method.delivery_tag)
        
        self.channel.basic_consume(
            queue='dlq',
            on_message_callback=callback,
            auto_ack=False
        )
        self.channel.start_consuming()
    
    def close(self):
        self.connection.close()


if __name__ == '__main__':
    dlq = DLQExample()
    dlq.setup_with_dlq()
    
    # 发送消息，5秒后会进入DLQ
    dlq.publish_to_main({'id': 1, 'message': 'This will expire'})
    
    print("Wait 5 seconds for message to expire and enter DLQ...")
    print("Then consume from DLQ:")
    # dlq.consume_from_dlq()  # 取消注释来消费DLQ消息
    
    dlq.close()
```

---

## Docker 部署 RabbitMQ

### 基本配置
```yaml
# docker-compose.yml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3.12-management-alpine
    container_name: rabbitmq
    ports:
      - "5672:5672"   # AMQP协议端口
      - "15672:15672" # 管理界面端口
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "-q", "ping"]
      interval: 30s
      timeout: 10s
      retries: 5

volumes:
  rabbitmq_data:
```

### 带持久化的配置
```yaml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3.12-management-alpine
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin_password
      RABBITMQ_VM_MEMORY_HIGH_WATERMARK: 0.8
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
      - ./rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf
    restart: unless-stopped
    ulimits:
      nofile:
        soft: 65536
        hard: 65536

volumes:
  rabbitmq_data:
```

### 集群配置
```yaml
version: '3.8'

services:
  rabbitmq-1:
    image: rabbitmq:3.12-management-alpine
    container_name: rabbitmq-1
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
      RABBITMQ_ERLANG_COOKIE: secret_cookie
    volumes:
      - rabbitmq1_data:/var/lib/rabbitmq

  rabbitmq-2:
    image: rabbitmq:3.12-management-alpine
    container_name: rabbitmq-2
    ports:
      - "5673:5672"
      - "15673:15672"
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
      RABBITMQ_ERLANG_COOKIE: secret_cookie
    volumes:
      - rabbitmq2_data:/var/lib/rabbitmq

volumes:
  rabbitmq1_data:
  rabbitmq2_data:
```

### 单容器快速启动
```bash
# 基本启动
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3.12-management-alpine

# 带持久化
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -v rabbitmq_data:/var/lib/rabbitmq \
  rabbitmq:3.12-management-alpine

# 带用户认证
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin123 \
  rabbitmq:3.12-management-alpine
```

### 常用 Docker 命令
```bash
# 启动RabbitMQ
docker start rabbitmq

# 停止RabbitMQ
docker stop rabbitmq

# 重启RabbitMQ
docker restart rabbitmq

# 进入RabbitMQ容器
docker exec -it rabbitmq bash

# 进入RabbitMQ管理CLI
docker exec -it rabbitmq rabbitmqctl bash

# 查看RabbitMQ状态
docker exec rabbitmq rabbitmqctl status

# 查看正在运行的容器
docker ps | grep rabbitmq

# 查看RabbitMQ日志
docker logs -f rabbitmq

# 带密码进入管理界面
# 访问 http://localhost:15672
# 默认账号: guest / guest
```

### RabbitMQ 配置文件示例
```ini
# rabbitmq.conf

# 默认用户和虚拟主机
default_vhost = /
default_user = guest
default_pass = guest

# 连接配置
listeners.tcp.default = 5672
management.tcp.port = 15672

# 内存和磁盘配置
vm_memory_high_watermark.relative = 0.8
disk_free_limit.absolute = 2GB

# 日志配置
log.console.level = info
log.file.level = info

# 性能配置
channel_max = 2048
heartbeat = 60
```

---

## MQ 日志查看

### RabbitMQ 容器日志

```bash
# 查看实时日志
docker logs -f rabbitmq

# 查看最近100行
docker logs --tail 100 rabbitmq

# 查看指定时间范围
docker logs --since "2024-01-01T00:00:00" rabbitmq

# 查找错误日志
docker logs rabbitmq 2>&1 | grep -i error

# 导出日志
docker logs rabbitmq > rabbitmq.log 2>&1
```

### RabbitMQ 管理界面

访问 `http://localhost:15672`（默认账号: guest/guest）

管理界面功能：
- **Overview**: 集群概览
- **Connections**: 连接信息
- **Channels**: 通道信息
- **Exchanges**: 交换机管理
- **Queues**: 队列管理
- **Admin**: 用户和权限管理

### RabbitMQ CLI 日志命令

```bash
# 进入容器
docker exec -it rabbitmq bash

# 查看集群状态
rabbitmqctl cluster_status

# 查看队列列表
rabbitmqctl list_queues

# 查看队列详情
rabbitmqctl list_queues name messages messages_ready messages_unacknowledged

# 查看交换机列表
rabbitmqctl list_exchanges

# 查看绑定关系
rabbitmqctl list_bindings

# 查看消费者
rabbitmqctl list_consumers

# 查看连接
rabbitmqctl list_connections

# 清除队列
rabbitmqctl purge_queue queue_name

# 关闭连接
rabbitmqctl close_connection <connection_pid> <reason>

# 查看用户列表
rabbitmqctl list_users

# 查看虚拟主机
rabbitmqctl list_vhosts
```

### 日志位置

容器内日志路径：
```bash
# 日志目录
/var/log/rabbitmq/

# 启动日志
/var/log/rabbitmq/startup_log
/var/log/rabbitmq/startup_err

# 交换日志
/var/log/rabbitmq/rabbit@localhost.log
```

```bash
# 容器内查看日志文件
docker exec rabbitmq cat /var/log/rabbitmq/rabbit@localhost.log

# 查看详细日志
docker exec rabbitmq cat /var/log/rabbitmq/startup_log
```

### 健康检查

```bash
# 容器健康检查
docker inspect rabbitmq | grep -A 10 "Health"

# RabbitMQ健康诊断
docker exec rabbitmq rabbitmq-diagnostics ping

# 检查端口
docker exec rabbitmq rabbitmq-diagnostics -q ports

# 检查内存使用
docker exec rabbitmq rabbitmqctl status | grep -A 5 "memory"
```
