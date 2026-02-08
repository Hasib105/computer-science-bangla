# RabbitMQ পরিচিতি

## 🎯 RabbitMQ কি?

**RabbitMQ** হলো একটি traditional message broker যা AMQP (Advanced Message Queuing Protocol) সাপোর্ট করে।

```
RabbitMQ Features:
├── Easy to use
├── Multiple protocols (AMQP, MQTT, STOMP)
├── Flexible routing
├── Acknowledgements
├── Management UI
└── Plugins ecosystem
```

## 🏗️ RabbitMQ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      RabbitMQ Broker                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐    ┌──────────────────┐    ┌──────────────────┐  │
│  │ Producer │───→│     Exchange     │───→│      Queue       │  │
│  └──────────┘    │                  │    │                  │  │
│                  │  ┌────────────┐  │    │  ┌────────────┐  │  │
│                  │  │ Routing    │  │    │  │ Messages   │  │  │
│                  │  │ Logic      │  │    │  │ [1][2][3]  │──┼──┼→ Consumer
│                  │  └────────────┘  │    │  └────────────┘  │  │
│                  └──────────────────┘    └──────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 📊 Exchange Types

### ১. Direct Exchange
```
Routing key exact match।

Exchange: "direct_logs"
                │
    ┌───────────┼───────────┐
    │           │           │
  "error"    "warning"    "info"
    │           │           │
    ↓           ↓           ↓
 Queue A     Queue B     Queue C
```

### ২. Topic Exchange
```
Pattern matching with * and #

"order.*"     → order.created, order.updated
"order.#"     → order.created, order.items.added

Exchange: "topic_logs"
                │
    ┌───────────┴───────────┐
    │                       │
"*.error"            "order.#"
    │                       │
    ↓                       ↓
 Queue A                Queue B
(all errors)          (all order events)
```

### ৩. Fanout Exchange
```
সব queue-তে broadcast।

Exchange: "fanout_logs"
                │
    ┌───────────┼───────────┐
    │           │           │
    ↓           ↓           ↓
 Queue A     Queue B     Queue C
 (gets all)  (gets all)  (gets all)
```

## 🔧 Implementation

### Producer
```python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

# Declare queue
channel.queue_declare(queue='task_queue', durable=True)

# Send message
channel.basic_publish(
    exchange='',
    routing_key='task_queue',
    body='Hello World!',
    properties=pika.BasicProperties(
        delivery_mode=2,  # Persistent
    )
)

connection.close()
```

### Consumer
```python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()

channel.queue_declare(queue='task_queue', durable=True)

def callback(ch, method, properties, body):
    print(f"Received: {body}")
    # Process message
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(
    queue='task_queue',
    on_message_callback=callback
)

channel.start_consuming()
```

## ⚖️ Kafka vs RabbitMQ

```
┌────────────────────┬────────────────────────────────────┐
│      RabbitMQ      │              Kafka                 │
├────────────────────┼────────────────────────────────────┤
│ Smart broker       │ Dumb broker, smart consumer       │
│ Push model         │ Pull model                         │
│ Message deletion   │ Log retention                      │
│ Flexible routing   │ Partitions only                    │
│ Lower throughput   │ Higher throughput                  │
│ Task queues        │ Event streaming                    │
│ Simple setup       │ Complex setup                      │
└────────────────────┴────────────────────────────────────┘

RabbitMQ: Task distribution, RPC
Kafka: Event streaming, log aggregation
```

---

🎉 মেসেজ কিউ সেকশন সম্পূর্ণ!

[মাইক্রোসার্ভিস →](../07-microservices/README.md)
