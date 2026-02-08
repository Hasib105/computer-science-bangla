# Apache Kafka পরিচিতি

## 🎯 Kafka কি?

**Apache Kafka** হলো একটি distributed event streaming platform যা high-throughput, fault-tolerant messaging এর জন্য ব্যবহৃত হয়।

```
Kafka Features:
├── High throughput (millions of messages/sec)
├── Fault tolerant (replication)
├── Scalable (partitioning)
├── Durable (disk persistence)
└── Real-time processing
```

## 🏗️ Kafka Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Kafka Cluster                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    ZooKeeper Cluster                      │  │
│  │          (Cluster coordination & metadata)                │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                  │
│         ┌────────────────────┼────────────────────┐            │
│         ↓                    ↓                    ↓            │
│    ┌─────────┐          ┌─────────┐          ┌─────────┐       │
│    │ Broker 1│          │ Broker 2│          │ Broker 3│       │
│    │         │          │         │          │         │       │
│    │ ┌─────┐ │          │ ┌─────┐ │          │ ┌─────┐ │       │
│    │ │P0-L │ │          │ │P0-F │ │          │ │P1-L │ │       │
│    │ │P1-F │ │          │ │P1-F │ │          │ │P2-F │ │       │
│    │ │P2-L │ │          │ │P2-F │ │          │ │P0-F │ │       │
│    │ └─────┘ │          │ └─────┘ │          │ └─────┘ │       │
│    └─────────┘          └─────────┘          └─────────┘       │
│                                                                 │
│    L = Leader partition, F = Follower (replica)                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 📊 Key Concepts

### Topic & Partitions
```
Topic: Logical channel for messages
Partitions: Topic divided for parallelism

Topic: "orders"
├── Partition 0: [msg1, msg2, msg3...]
├── Partition 1: [msg4, msg5, msg6...]
└── Partition 2: [msg7, msg8, msg9...]

Messages in a partition are ordered.
Messages across partitions have no order guarantee.
```

### Producers
```python
from kafka import KafkaProducer
import json

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

# Send message
producer.send('orders', {
    'order_id': 123,
    'user_id': 456,
    'amount': 1000
})

# With key (ensures same key goes to same partition)
producer.send('orders', 
    key=b'user_456',  # Partition key
    value={'order_id': 123}
)
```

### Consumers & Consumer Groups
```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'orders',
    bootstrap_servers=['localhost:9092'],
    group_id='order-processor',
    auto_offset_reset='earliest'
)

for message in consumer:
    order = json.loads(message.value)
    process_order(order)
```

### Consumer Group Scaling
```
Topic: orders (3 partitions)
Consumer Group: order-processor

Scenario 1: 1 Consumer
Consumer A reads from P0, P1, P2

Scenario 2: 3 Consumers
Consumer A → P0
Consumer B → P1
Consumer C → P2

Scenario 3: 5 Consumers
Consumer A → P0
Consumer B → P1
Consumer C → P2
Consumer D → Idle
Consumer E → Idle
(Consumers > Partitions = some idle)
```

## ⚡ When to Use Kafka

```
✓ High-throughput event streaming
✓ Log aggregation
✓ Real-time analytics
✓ Event sourcing
✓ Microservices communication
✓ Data pipelines

✗ Simple task queues (use RabbitMQ)
✗ Low latency requirements (< 10ms)
✗ Small scale applications
```

## 📚 পরবর্তী টপিক

[RabbitMQ পরিচিতি →](./04-rabbitmq.md)
