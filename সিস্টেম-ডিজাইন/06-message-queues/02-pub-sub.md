# Pub/Sub প্যাটার্ন

## 🎯 Pub/Sub কি?

**Publish/Subscribe (Pub/Sub)** হলো একটি মেসেজিং প্যাটার্ন যেখানে publisher মেসেজ পাঠায় এবং সব subscriber সেই মেসেজ পায়।

```
Traditional Point-to-Point:
Producer → Queue → Consumer (একজন পায়)

Pub/Sub:
Publisher → Topic → All Subscribers (সবাই পায়)

                              ┌────────────┐
                         ┌───→│Subscriber A│
┌──────────┐    ┌───────┐│    └────────────┘
│Publisher │ ─→ │ Topic ├┤    ┌────────────┐
└──────────┘    └───────┘├───→│Subscriber B│
                         │    └────────────┘
                         │    ┌────────────┐
                         └───→│Subscriber C│
                              └────────────┘
```

## 📊 Pub/Sub vs Message Queue

```
┌────────────────────┬────────────────────────────────────┐
│    Message Queue   │           Pub/Sub                  │
├────────────────────┼────────────────────────────────────┤
│ একজন consumer     │ সব subscriber পায়                 │
│ Work distribution  │ Event broadcasting                 │
│ Task processing    │ Notifications, Updates            │
│ Load balancing     │ Fan-out pattern                   │
└────────────────────┴────────────────────────────────────┘
```

## 🔄 Pub/Sub Use Cases

### ১. Event Broadcasting
```
User signs up → Publish "user.created" event

Subscribers:
├── Email Service → Welcome email
├── Analytics → Track signup
├── Referral → Check referral code
└── Notification → In-app notification
```

### ২. Real-time Updates
```
Price change → Publish to topic

Subscribers:
├── Mobile App Users
├── Web App Users
└── Partner APIs
```

### ৩. Microservices Communication
```
Order created:

Order Service → "order.created" topic
                      │
    ┌─────────────────┼─────────────────┐
    ↓                 ↓                 ↓
Payment          Inventory        Notification
Service          Service           Service
```

## 🏗️ Implementation Example

### Redis Pub/Sub
```python
# Publisher
import redis

r = redis.Redis()
r.publish('notifications', 'New order received!')

# Subscriber
pubsub = r.pubsub()
pubsub.subscribe('notifications')

for message in pubsub.listen():
    print(message['data'])
```

### Kafka Topics
```python
# Producer
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')
producer.send('user-events', b'User created')

# Consumer (with consumer group)
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'user-events',
    group_id='notification-service',
    bootstrap_servers='localhost:9092'
)

for message in consumer:
    print(message.value)
```

## ⚡ Consumer Groups

```
Consumer group দিয়ে scaling:

Topic: orders
├── Partition 0 ─→ Consumer A (Group: payment)
├── Partition 1 ─→ Consumer B (Group: payment)
└── Partition 2 ─→ Consumer C (Group: payment)

আবার আলাদা group:
├── Partition 0 ─→ Consumer X (Group: analytics)
├── Partition 1 ─→ Consumer Y (Group: analytics)
└── Partition 2 ─→ Consumer Z (Group: analytics)

Each group receives ALL messages.
Within a group, messages are distributed.
```

## 💡 Best Practices

```
✓ Topic naming convention (domain.event)
✓ Message schema versioning
✓ Dead letter queue for failures
✓ Idempotent subscribers
✓ Monitoring & alerting
✓ Retention policy
```

## 📚 পরবর্তী টপিক

[Kafka পরিচিতি →](./03-kafka.md)
